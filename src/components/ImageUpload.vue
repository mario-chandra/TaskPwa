
<script setup>

    import { ref } from 'vue';

    const reader = new FileReader();
    const convertBlob = () => {
        const data = 
        fetch("https://picsum.photos/id/222/300/300.jpg")
        .then((response) => {
            return response.blob();
        })
        .then((blob) => {
             return blob;
        });

        return data   
    }

    const dbName = "the_name";
    let datas = await convertBlob()
    console.log(datas)
    let blobs = ref([]);
    let images64 = ref();

    let image = new Image();
    image.src = images64;

    const convertBlobToBase64 = async (blob) => {
    return await blobToBase64(blob);
    }

    const blobToBase64 = blob => new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(blob);
    reader.onload = () => resolve(reader.result);
    reader.onerror = error => reject(error);
    });


    images64 = await convertBlobToBase64(datas);

    const addData = () => {
    const request = indexedDB.open(dbName, 3);
  
    request.onerror = (event) => {
      console.error("Error opening database:", event.target.error);
    };
  
    request.onupgradeneeded = (event) => {
      const db = event.target.result;
  
      const objectStore = db.createObjectStore("blob", { keyPath: "size"});
      objectStore.createIndex("size", "size", { unique: false });
      objectStore.createIndex("type", "type", { unique: false });
    };
  
    request.onsuccess = (event) => {
      const db = event.target.result;
  
      const addTransaction = db.transaction("blob", "readwrite");
      const blobObjectStore = addTransaction.objectStore("blob");
  
      const addRequest = blobObjectStore.put(datas);
  
      addRequest.onsuccess = (event) => {
        console.log("Data added successfully");
      };
  
      addRequest.onerror = (event) => {
        console.error("Error adding data", event.target.error);
      };
  
      addTransaction.oncomplete = () => {
        console.log("Add transaction completed");
        db.close();
      };
      readData();
    };
  };


  const readData = () => {
    const request = indexedDB.open(dbName, 3);
    blobs.value = [];
    request.onerror = (event) => {
      console.error("Error opening database:", event.target.error);
    };
  
    request.onsuccess = (event) => {
      const db = event.target.result;
  
      const readTransaction = db.transaction("blob", "readonly");
      const blobObjectStore = readTransaction.objectStore("blob");
  
      const blobsCursor = blobObjectStore.openCursor();
  
      blobsCursor.onsuccess = (event) => {
        const cursor = event.target.result;
        if (cursor) {
            blobs.value.push(cursor.value);
          cursor.continue();
        } else {
          console.log("Data read successfully");
          db.close();
        }
      };
  
      customersCursor.onerror = (event) => {
        console.error("Error reading data", event.target.error);
        db.close();
      };
    };
  };

  addData()
</script>

<template>
    <button @click="readData">Read Data</button>
    <img :src="images64" v-bind:alt="tests"> 
    <ul>
        <li v-for="blob in blobs" :key="blob.size">
            {{ blob.size }} - {{ blob.type }}
        </li>
    </ul>
</template>