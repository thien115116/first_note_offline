<template>
  <div>
    <label for="name">Name:</label>
    <input id="name" v-model="name">
  </div>
</template>

<script>
import { openDB } from 'idb';

const dbPromise = openDB('my-db', 1, {
  upgrade(db) {
    db.createObjectStore('inputs');
  }
});

export default {
  data() {
    return {
      name: ''
    };
  },
  mounted() {
    dbPromise.then(db => {
      return db.get('inputs', 'name');
    }).then(value => {
      if (value) {
        this.name = value;
      }
    });
  },
  watch: {
    name(newValue) {
      dbPromise.then(db => {
        return db.put('inputs', newValue, 'name');
      });
    }
  },
  created() {
    window.addEventListener('offline', () => {
      // Set a flag in IndexedDB indicating the app is offline
      dbPromise.then(db => {
        return db.put('inputs', 'true', 'offline');
      });
    });
    window.addEventListener('online', () => {
      // Remove the flag from IndexedDB when the app is online again
      dbPromise.then(db => {
        return db.delete('inputs', 'offline');
      });
      // Update the input values from IndexedDB
      dbPromise.then(db => {
        return db.get('inputs', 'name');
      }).then(value => {
        if (value) {
          this.name = value;
        }
      });
    });
  }
};
</script>
