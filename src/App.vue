<template>
  <div id="app" class="container">
    <div class="column">
      <div v-if="state === 'synced'" class="notification is-success">
      Form is synced with Firestore
    </div>
    <div v-if="state === 'modified'" class="notification is-info">
      Form data changed, will sync with Firebase
    </div>
    <div v-if="state === 'error'" class="notification is-danger">
      Failed to save to Firestore. {{ errorMessage }}
    </div>
    <div v-if="state === 'revoked'" class="notification is-warning">
      Form reverted to original data.
    </div>
    </div>

    <div class="column">
      <form @submit.prevent="updateFirebase" @input="fieldUpdate">
      <div class="field">
        <input type="text" name="name" v-model="formData.name" class="input" placeholder="Name">
      </div>
      <div class="field">
        <input type="email" name="name" v-model="formData.email" class="input" placeholder="Email">
      </div>
      <div class="field">
        <input type="tel" name="name" v-model="formData.tel" class="input" placeholder="tel">
      </div>
      <div class="field is-grouped">
        <div class="control">
          <button type="submit" class="button is-link">Submit</button>
        </div>
      </div>
    </form>
    </div>
    <div class="column">
      <button @click="revertToOriginal" class="button is-warning is-rounded">
      <span class="icon is-small">
        <i class="fas fa-history"></i>
      </span>
      <span>Revert to original data</span>
    </button>
    </div>
  </div>
</template>

<script>
import { db } from './assets/firebase';

import { debounce } from 'debounce';

const documentPath = 'contacts/a423aa';
export default {
  data() {
    return {
      firebaseData: null,
      formData: {},
      state: 'loading',
      errorMessage: '',
      originalData: null,
    }
  },

  firestore() {
    return {
      firebaseData: db.doc(documentPath),
    }
  },

  methods: {
    async updateFirebase() {
      try {
        await db.doc(documentPath).set(this.formData);
        this.state = 'synced';
      } catch (error) {
        this.errorMessage = JSON.stringify(error);
        this.state = 'error';
      }
    },

    fieldUpdate() {
      this.state = 'modified';
      this.debouncedUpdate();
    },

    debouncedUpdate: debounce(function() {
      this.updateFirebase()
    }, 1500),

    revertToOriginal() {
      this.state = 'revoked';
      this.formData = { ...this.originalData }
    }
  },

  created: async function () {
    const docRef = db.doc(documentPath);
    
    let data = (await docRef.get() ).data();

    if (!data) {
      data = {name: '', tel: '', email: ''}
      docRef.set(data)
    }

    this.formData = data;

    this.originalData = { ...data };

    this.state = 'synced';
  }
}
</script>