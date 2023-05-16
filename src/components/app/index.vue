<style src="../../styles/app.scss"></style>

<template>
  <form v-if="defaultEmail != null" :action="actionUrl" method="post" target="_blank"
        class="flex-columns-container height-fill">
    <!-- <div class="form-group">
      <label for="email" class="form-label">{{ label1 }}</label>
      <input id="email" type="email" class="form-control" name="email" placeholder="example@contoso.com"
             v-model="defaultEmail" />
    </div>
    <div class="form-group height-fill flex-columns-container">
      <label for="message" class="form-label">{{ label2 }}</label>
      <textarea id="message" class="form-control flex-grow" name="message" :placeholder="placeholder"></textarea>
    </div> -->

    <label class="form-label">API List</label>
    <div class="form-group height-fill flex-columns-container">
      <ul>
      <li v-for="item in items" :key="item.id">{{ item.name }}</li>
      </ul>
    </div>

    <!-- <div class="form-group">
      <button type="submit" class="button button-primary">Submit</button>
    </div> -->

    <div>
      <button @click="fetchData">Fetch Data</button>
      <label class="form-label">{{apiResponse}}</label>
    </div>

    <div class="form-group">
      <label class="form-label">Name</label>
      <input id="name" type="text" class="form-control" v-model="inputName" />
      <button type="submit" @click="postData">Post Data</button>
      <label class="form-label">{{postDataResponse}}</label>
    </div>

  </form>
  <div v-else class="loading"></div>
</template>

<script lang="ts">
import {getValues} from "@azure/api-management-custom-widgets-tools"
import {valuesDefault} from "../../values"
import axios from "axios"

export default {
  data() {
    return {
      label1: null,
      label2: null,
      placeholder: null,
      actionUrl: null,
      defaultEmail: null,
      items: [],
      apiResponse: null,
      postDataResponse: null,
      inputName: ''
    }
  },

  inject: ["secretsPromise", "requestPromise"],

  async mounted(): Promise<void> {
    const editorData = getValues(valuesDefault)
    this.label1 = editorData.label1
    this.label2 = editorData.label2
    this.placeholder = editorData.placeholder
    this.actionUrl = editorData.actionUrl
    

    const [secrets, request] = await Promise.all([this.secretsPromise, this.requestPromise])

    if (!secrets.userId) {
      this.defaultEmail = ""
      return
    }

    request(`/users/${secrets.userId}`)
      .then(e => {
        e.json()
        console.log("e",e) 
      console.log("e.json",e.json()) })
      .then(({properties}) => this.defaultEmail = properties.email)
      .catch(e => {
        console.error("Could not prefill the email address!", e)
        this.defaultEmail = ""
      })

    request(`/apis`)
      .then((response) => {
          if (!response.ok) {
            throw Error("Error fetching items");
          }
          return response.json();
        })
        .then((data) => {
          this.items = data.value;
        })
        .catch((error) => {
          console.log(error);
        });

    },

    methods: {

     async fetchData() {

        const [secrets1] = await Promise.all([this.secretsPromise])
        const token = `${secrets1.token}`; // Replace with your actual token

        const axiosInstance = axios.create({
          headers: {
            Authorization: `Bearer ${token}`,
          },
        });

        axiosInstance.get('https://backend-bban.azurewebsites.net/api/check-your-backend')
          .then(response => {
            // Handle the response
            console.log(response.data);
            this.apiResponse=response.data;
          })
          .catch(error => {
            // Handle the error
            console.error('Error fetching data:', error);
          });
      },

      async postData() {

      const [secrets1] = await Promise.all([this.secretsPromise])
      const token = `${secrets1.token}`; // Replace with your actual token

      const axiosInstance = axios.create({
        headers: {
          Authorization: `Bearer ${token}`,
        }
      });

      const requestBody = {
        "name": this.inputName
      };

      axiosInstance.post('https://backend-bban.azurewebsites.net/api/post-data-backend?code=aMettnDqRZQcCE4JvwBx-zBQsmZ4lxxMwH7kV0JFwttoAzFuFAKH8Q==', requestBody)
        .then(response => {
          // Handle the response
          console.log(response.data);
          this.postDataResponse=response.data;
        })
        .catch(error => {
          // Handle the error
          console.error('Error fetching data:', error);
        });
      },
    },
}
</script>
