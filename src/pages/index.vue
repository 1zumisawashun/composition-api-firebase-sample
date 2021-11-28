<template>
  <div class="container">
    <div>
      <Logo />
      <h1 class="title">nuxt-firebase</h1>
      <div class="links">
        <a
          href="https://nuxtjs.org/"
          target="_blank"
          rel="noopener noreferrer"
          class="button--green"
        >
          Documentation
        </a>
        <a
          href="https://github.com/nuxt/nuxt.js"
          target="_blank"
          rel="noopener noreferrer"
          class="button--grey"
        >
          GitHub
        </a>
        <button
          target="_blank"
          rel="noopener noreferrer"
          class="button--grey"
          @click="sayHello"
        >
          Hello
        </button>
        <button
          target="_blank"
          rel="noopener noreferrer"
          class="button--grey"
          @click="handleFirestore"
        >
          Firestore
        </button>
        <button
          target="_blank"
          rel="noopener noreferrer"
          class="button--grey"
          @click="handleStorage"
        >
          storage
        </button>
        <input type="file" @change="onFileChange" />
        <img :src="uploadedImage" alt="" width="100" />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "@vue/composition-api";
import { functions, firestore, storage } from "../plugins/firebase";

export default defineComponent({
  setup() {
    const sayHello = () => {
      const data: String = "NuxtCompositionAPI";
      console.log(`Hello,${data}`);
    };
    const put = (file: any): Promise<string> => {
      return new Promise((resolve, reject) => {
        // ファイルのパスを生成
        const storageRef = storage.ref();
        // const mountainsRef = storageRef.child(`insects/${file.name}`)
        const mountainsRef = storageRef.child(`file.png`);

        // ファイルをアップロード
        mountainsRef
          .put(file)
          .then((snapshot) => {
            // アップロードしたファイルのURLを取得
            snapshot.ref
              .getDownloadURL()
              .then((downloadURL: string) => {
                resolve(downloadURL);
              })
              .catch((error) => {
                reject(error.message);
              });
          })
          .catch((error) => {
            reject(error.message);
          });
      });
    };
    const handleFirestore = async () => {
      try {
        firestore.collection("post").add({
          test: "composition API test",
        });
        const postRef = await firestore.collection("post");
        const result = await postRef.get();
        console.log(result, "result");
      } catch (error) {
        console.error(error);
      }
    };
    const handleStorage = async () => {
      const result = await put(file.value);
      console.log(result, "result");
    };

    // 画像ファイルをBase64に変換
    const getBase64 = (file: File): Promise<string | ArrayBuffer | null> => {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = () => resolve(reader.result);
        reader.onerror = (error) => reject(error);
        reader.readAsDataURL(file);
      });
    };

    const uploadedImage = ref<string | ArrayBuffer | null>(null);
    const file = ref<File | undefined>(undefined);

    const onFileChange = (e: Event) => {
      if (e.target instanceof HTMLInputElement && e.target.files) {
        file.value = e.target.files[0];
        getBase64(e.target.files[0]).then(
          (image: string | ArrayBuffer | null) => {
            uploadedImage.value = image;
          }
        );
      }
    };

    return {
      handleFirestore,
      handleStorage,
      onFileChange,
      uploadedImage,
      file,
      sayHello,
    };
  },
});
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
