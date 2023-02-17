<template>
  <div class="form-group">
    <label for="name" class="mb-2">Name:</label>
    <input class="form-control" id="name" v-model="conservationReports.name" />
  </div>
  <div class="form-group mt-3">
    <label for="phoneNumber" class="mb-2">Phone Number:</label>
    <input
      class="form-control"
      id="phoneNumber"
      type="text"
      v-model="conservationReports.phoneNumber"
    />
  </div>
  <div class="form-group mt-3">
    <label for="file" class="mb-2">File</label>
    <input
      class="form-control"
      id="file"
      type="file"
      accept="image/png, image/gif, image/jpeg"
      multiple
      @change="fileChange"
    />
  </div>
  <div>
    <img :src="`data:image/png;base64,${image}`" alt="" v-for="(image, index) in images" :key="index">
  </div>
</template>

<script>
import { openDB } from "idb";

const dbPromise = openDB("valqua-spm", 1, {
  upgrade(db) {
    db.createObjectStore("conservationReports");
  },
});

export default {
  data() {
    return {
      conservationReports: {
        name: "",
        phoneNumber: "",
        files: [],
      },
      images: []
    };
  },
  mounted() {
    dbPromise
      .then((db) => {
        return db.get("conservationReports", "conservationReports");
      })
      .then((value) => {
        if (value) {
          this.conservationReports = JSON.parse(value);
          this.images = this.conservationReports.files;
        }
      });
  },
  watch: {
    conservationReports: {
      handler() {
        dbPromise.then((db) => {
          return db.put(
            "conservationReports",
            JSON.stringify(this.conservationReports),
            "conservationReports"
          );
        });
      },
      deep: true,
    },
  },
  created() {
    window.addEventListener("offline", () => {
      // Set a flag in IndexedDB indicating the app is offline
      dbPromise.then((db) => {
        return db.put("conservationReports", "true", "offline");
      });
    });
    window.addEventListener("online", () => {
      // Remove the flag from IndexedDB when the app is online again
      dbPromise.then((db) => {
        return db.delete("conservationReports", "offline");
      });
      // Update the input values from IndexedDB
      dbPromise
        .then((db) => {
          return db.get("conservationReports", "conservationReports");
        })
        .then((value) => {
          if (value) {
            this.conservationReports = JSON.parse(value);
          }
        });
    });
  },
  methods: {
    async fileChange(e) {
      const files = e.target.files;
      if (!files) return;
      Array.from(files).forEach(async(file) => {
        const base64 = await this.convertToBase64(file);
        this.images.push((base64));
        this.conservationReports.files.push(base64)
      });
    },
    convertToBase64(file) {
      return new Promise((resolve, reject) => {
    const reader = new FileReader();

    reader.addEventListener('load', () => {
      // Convert the image to a Base64 encoded string
      const base64String = reader.result.replace(/^data:image\/(png|jpg|jpeg);base64,/, '');
      resolve(base64String); // Resolve the promise with the Base64 string
    });

    reader.addEventListener('error', () => {
      reject(reader.error); // Reject the promise with the error
    });

    reader.readAsDataURL(file);
  });
    },
  },
};
</script>
