<template>
  <v-dialog max-width="600px" v-model="dialog">
    <v-btn flat slot="activator" class="success">Add New Activity</v-btn>
    <v-card>
      <v-card-title>
        <h2>Add a New Activity</h2>
      </v-card-title>
      <v-card-text>
        <v-form class="px-3" ref="form">
          <v-text-field v-model="title" label="Title" prepend-icon="folder" :rules="inputRules"></v-text-field>
          <v-textarea v-model="content" label="Information" prepend-icon="edit" :rules="inputRules"></v-textarea>

          <v-menu v-model="menu" :close-on-content-click="false">
            <v-text-field
              slot="activator"
              :rules="inputRules"
              :value="formattedDate"
              clearable
              label="Due date"
              prepend-icon="date_range"
            ></v-text-field>
            <v-date-picker v-model="due" @change="menu = false"></v-date-picker>
          </v-menu>

          <v-spacer></v-spacer>

          <v-btn flat @click="submit" class="success mx-0 mt-3">Add Activity</v-btn>
        </v-form>
      </v-card-text>
    </v-card>
  </v-dialog>
</template>

<script>
import format from "date-fns/format";
import db from "@/fb";
import firebase from "firebase";

export default {
  data() {
    return {
      title: "",
      content: "",
      due: null,
      menu: false,
      inputRules: [
        v => !!v || "This field is required",
        v => v.length >= 3 || "Minimum length is 3 characters"
      ],
      dialog: false,
      person: ""
    };
  },
  mounted() {
    var self = this;
    firebase.auth().onAuthStateChanged(function(user) {
      if (user) {
        // User is signed in.
        self.person = user.displayName;
      } else {
        // No user is signed in.
        console.log("no user hello");
      }
    });
  },
  methods: {
    submit() {
      var self = this;
      if (this.$refs.form.validate()) {
        this.loading = true;

        const todo = {
          title: this.title,
          content: this.content,
          due: format(this.due, "Do MMM YYYY"),
          person: this.person,
          status: "ongoing"
        };

        db.collection("masterProjectList")
          .doc(this.$store.state.selectedProject)
          .collection("todos")
          .add(todo)
          .then(() => {
            this.dialog = false;
            this.$emit("projectAdded");
          });
      }
    }
  },
  computed: {
    formattedDate() {
      console.log(this.due);
      return this.due ? format(this.due, "Do MMM YYYY") : "";
    }
  }
};
</script>
