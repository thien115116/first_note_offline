<template>
  <div class="mb-3 d-flex">
    <label
      :class="[`btn btn-${status === 'online' ? 'primary' : 'secondary'}`]"
    >
      You are {{ status }} now
    </label>
    <button class="btn btn-warning ms-3" @click="resetIndexDb">Reset</button>
  </div>

  <h3 class="text-start">Offline Notes</h3>
  <div class="form-group text-start">
    <label for="name" class="mb-2">Name:</label>
    <input class="form-control" id="name" v-model="conservationReports.name" />
  </div>
  <div class="form-group mt-3 text-start">
    <label for="phoneNumber" class="mb-2">Phone Number:</label>
    <input
      class="form-control"
      id="phoneNumber"
      type="text"
      v-model="conservationReports.phoneNumber"
    />
  </div>
  <div class="d-flex mt-3 text-start">
<div class="d-flex">
      <label for="sex_boy" class="me-2">Boy</label>
      <input
      id="sex_boy"
        class="form-check"
        name="sex"
        type="radio"
        :checked="conservationReports.sex === 'boy' ? 'checked':''"
        @change="changeSex('boy')"
      />
</div>
<div class="ms-3 d-flex">
      <label for="sex_girl" class="me-2">Girl</label>
      <input
      id="sex_girl"
        class="form-check"
        name="sex"
        type="radio"
        :checked="conservationReports.sex === 'girl' ? 'checked':''"
        @change="changeSex('girl')"
      />
</div>
  </div>
  <div class="form-group mt-3 text-start">
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
    <img
      :src="`data:image/png;base64,${image}`"
      alt=""
      v-for="(image, index) in images"
      class="img-fluid"
      :key="index"
    />
  </div>
</template>

<script>
import { openDB } from "idb";
const objectStoreName = Date.now();

function setQueryStringParameter(name, value) {
    const params = new URLSearchParams(window.location.search);
    params.set(name, value);
    window.history.replaceState({}, "", decodeURIComponent(`${window.location.pathname}?${params}`));
}


export default {
  data() {
    return {
      conservationReports: {
        name: "",
        phoneNumber: "",
        files: [],
      },
      images: [],
      status: "online",
      pageId:''
    };
  },
  // mounted() {
  //   this.dbPromise
  //     .then((db) => {
  //       return db.get(this.pageId, "conservationReports");
  //     })
  //     .then((value) => {
  //       if (value) {
  //         console.log(value);
  //         this.conservationReports = JSON.parse(value);
  //         this.images = this.conservationReports.files;
  //       }
  //     });
  // },
  watch: {
    conservationReports: {
      handler() {
        this.dbPromise.then((db) => {
          return db.put(
            this.pageId,
            JSON.stringify(this.conservationReports),
            "conservationReports"
          );
        });
      },
      deep: true,
    },
  },
  async created() {
    // this.pageId = new URLSearchParams(window.location.search).get('pageId');
    await this.initDB();
    this.dbPromise
      .then((db) => {
        return db.get(this.pageId, "conservationReports");
      })
      .then((value) => {
        if (value) {
          console.log(value);
          this.conservationReports = JSON.parse(value);
          this.images = this.conservationReports.files;
        }
      });

    window.addEventListener("offline", () => {
      this.status = "offline";
      // Set a flag in IndexedDB indicating the app is offline
      this.dbPromise.then((db) => {
        return db.put(this.pageId, "true", "offline");
      });
    });
    window.addEventListener("online", () => {
      this.status = "online";
      // Remove the flag from IndexedDB when the app is online again
      this.dbPromise.then((db) => {
        return db.delete(this.pageId, "offline");
      });
      // Update the input values from IndexedDB
      this.dbPromise
        .then((db) => {
          return db.get(this.pageId, "conservationReports");
        })
        .then((value) => {
          if (value) {
            console.log(value);
            this.conservationReports = JSON.parse(value);
          }
        });
    });
  },
  methods: {
    initDB() {
      return new Promise((resolve) => {
        const pageId = new URLSearchParams(window.location.search).get('pageId');
        if (pageId) {
          this.pageId = pageId
        } else {
          this.pageId = objectStoreName
          setQueryStringParameter("pageId", this.pageId)
        }

        this.dbPromise = openDB("valqua-spm", Number(this.pageId), {
          upgrade(db) {
            console.log('111')
            if (!db.objectStoreNames.contains(self.pageId)) {
              console.log('222')
              db.createObjectStore(self.pageId);
            }
            resolve()
          },
        });

        const self = this;

        console.log('pageId', pageId)
        // const request = indexedDB.open('valqua-spm', Date.now());
        // request.onsuccess = function(event) {
        //   console.log('333')
        //   const db = event.target.result;
        //   console.log(`Database connection successful for ${self.pageId}`);
        //   // Use the database connection and object store here
        //   console.log(db);
        //   // if (!pageId) {
        //   //   setQueryStringParameter("pageId", self.pageId)
        //   // }
        //   resolve()
        // };
      })
    },
    async fileChange(e) {
      const files = e.target.files;
      if (!files) return;
      Array.from(files).forEach(async (file) => {
        console.log(this);
        const base64 = await this.convertToBase64(file);
        this.images?.push(base64);
        this.conservationReports?.files?.push(base64);
      });
    },
    convertToBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();

        reader.addEventListener("load", () => {
          // Convert the image to a Base64 encoded string
          const base64String = reader.result.replace(
            /^data:image\/(png|jpg|jpeg);base64,/,
            ""
          );
          resolve(base64String); // Resolve the promise with the Base64 string
        });

        reader.addEventListener("error", () => {
          reject(reader.error); // Reject the promise with the error
        });

        reader.readAsDataURL(file);
      });
    },
    resetIndexDb() {
      this.dbPromise.then((db) => {
        return db.clear(this.pageId);
      });
      this.conservationReports = { name: "", phoneNumber: "", files: [] };
      caches.keys().then(function (names) {
        for (let name of names) caches.delete(name);
      });
    },
    changeSex(sex) {
      this.conservationReports.sex = sex;
    }
  },
};
</script>
