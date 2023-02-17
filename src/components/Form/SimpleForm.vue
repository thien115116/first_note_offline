<template>
  <div class="form-group">
    <label for="name">Name:</label>
    <input class="form-control" id="name" v-model="conservationReports.name" />
  </div>
  <div class="form-group">
    <label for="name">Phone Number:</label>
    <input
      class="form-control"
      id="phoneNumber"
      type="text"
      v-model="conservationReports.phoneNumber"
    />
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
      },
    };
  },
  mounted() {
    dbPromise.then(db => {
      return db.get('conservationReports', 'conservationReports');
    }).then(value => {
      if (value) {
        this.conservationReports = JSON.parse(value);
      }
    });
  },
  watch: {
    conservationReports: {
      handler() {
        console.log(this.conservationReports);
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
          return db.get("conservationReports", 'conservationReports');
        })
        .then((value) => {
          if (value) {
            this.conservationReports = JSON.parse(value);
          alert('Go Online Please Submit Form')
        }
        });
    });
  },
};
</script>
