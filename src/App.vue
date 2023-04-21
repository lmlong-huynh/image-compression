<template>
  <main>
    <div>
      <input
        type="file"
        accept="image/png, image/jpeg, image/jpg, image/heic"
        v-on:change="doSomethingCool($event.target.files[0])"
      />
    </div>
    <div style="display: flex; flex-wrap: wrap">
      <div style="margin-right: 10px; flex: 1" v-for="(img, i) in images" :key="i">
        <p>Type: {{ img.type }}, Size: {{ Math.round(img.size * 0.001) }} kB</p>
        <img style="width: 500px" :src="img.src" alt="" />
      </div>
    </div>
  </main>
</template>

<script setup>
import heic2any from 'heic2any';
import imageCompression from 'browser-image-compression';
import { ref } from 'vue';

const images = ref([]);

const isTypeHeic = (blob) => {
  return blob.type === 'image/heic';
};

const isValidImageSize = async (blob) => {
  const maxKb = 200;
  return Math.round(blob.size * 0.001) <= maxKb;
};

const heicToJpeg = async (blob) => {
  const jpeg = await heic2any({
    blob,
    toType: 'image/jpeg'
  });

  return jpeg;
};

const jpegCompress = async (jpeg, maxSizeMB) => {
  const compressionOptions = {
    maxSizeMB: maxSizeMB ?? 0.1,
    useWebWorker: true
  };

  try {
    console.log('try to compress');

    return await imageCompression(jpeg, compressionOptions);
  } catch (error) {
    console.log('error');
    return jpeg;
  }
};

const createObjectURL = (img) => {
  return URL.createObjectURL(img);
};

const compress = async (blob, maxSizeMB) => {
  console.log('start...');
  console.log(blob);

  if (!isValidImageSize(blob)) return;
  console.log(isTypeHeic(blob));
  const jpeg = isTypeHeic(blob) ? await heicToJpeg(blob) : blob;
  return await jpegCompress(jpeg, maxSizeMB);
};

const doSomethingCool = async (blob) => {
  console.log(isTypeHeic(blob));

  images.value = [];
  images.value.push(isTypeHeic(blob) ? await heicToJpeg(blob) : blob);

  const maxSizeMBs = [0.5, 0.4, 0.3, 0.2, 0.1];
  for (const size of maxSizeMBs) {
    const compressedImg = await compress(blob, size);
    console.log(compressedImg);
    images.value.push(compressedImg);
  }

  images.value = images.value.map((x) => {
    x.src = createObjectURL(x);
    return x;
  });
};
</script>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
