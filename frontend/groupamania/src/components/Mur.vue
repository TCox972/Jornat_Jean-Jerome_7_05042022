<template>
  <div class="main">
    <v-card
      class="mx-auto rounded-lg mt-16 mb-16 pb-3"
      elevation="7"
      max-height="650"
      max-width="550"
    >
      <div class="d-flex pt-4 ml-5">
        <p class="name font-weight-bold">{{ me.username }}</p>
      </div>
      <v-card-title>
        <v-spacer></v-spacer>
        Créer un nouveau message :
        <v-spacer></v-spacer>
        <v-btn icon @click="show = !show">
          <v-icon x-large color="red">{{
            show ? "mdi-note-off-outline" : "mdi-note-plus-outline"
          }}</v-icon>
        </v-btn>
        <v-spacer></v-spacer>
      </v-card-title>
      <v-expand-transition>
        <div v-show="show">
          <v-text-field
            v-model="title"
            :rules="[rules.required]"
            label="Titre"
            outlined
            class="px-7"
            required
          ></v-text-field>
          <v-textarea
            v-model="content"
            :rules="[rules.required]"
            name="message"
            outlined
            label="Message"
            auto-grow
            class="px-7"
            value=""
            required
          ></v-textarea>
          <v-file-input
            v-model="attachment"
            label="Image"
            prepend-icon="mdi-camera"
            class="px-7"
            accept="image/.jpg,.jpeg,.png,.gif"
          ></v-file-input>
          <div class="mr-7 text-right">
            <v-btn @click="publication" color="blue" text> Publier </v-btn>
          </div>
        </div>
      </v-expand-transition>
    </v-card>
    <v-card
      class="mx-auto rounded mt-8 mb-8"
      elevation="7"
      max-height="650"
      max-width="550"
      v-for="message in messages"
      :key="message.id"
    >
      <div class="headpost pt-4">
        <p class="name font-weight-bold">{{ message.User.username }}</p>
        <v-btn
          v-if="message.UserId == me.id || me.isAdmin == true"
          icon
          x-small
          @click="confirmdelete(message)"
        >
          <v-icon color="grey">mdi-delete-outline</v-icon>
        </v-btn>
      </div>
    
      <v-img
        class="white--text align-end"
        max-height="300px"
        :src="message.attachment"
      >
        <v-card-title class="title">{{ message.title }}</v-card-title>
      </v-img>

      <v-card-text class="text--primary">
        <div>{{ message.content }}</div>

        <div class="date text-right">
          {{ message.createdAt.slice(0, 10).split("-").reverse().join("/") }}
        </div>
      </v-card-text>
    </v-card>
  </div>
</template>

<script>
export default {
  name: "MurComp",

  data: () => ({
    messages: [],
    me: [],
    title: "",
    content: "",
    attachment: [],
    show: false,
    showalert: false,
    rules: {
      required: (value) => !!value || "Requis",
    },
  }),
  created() {
    const token = this.$store.state.user.token;
    fetch("http://localhost:3000/api/message/", {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    })
      .then((data) => data.json())
      .then((res) => {
        this.messages = res;
      });

    fetch("http://localhost:3000/api/auth/myaccount/", {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    })
      .then((data) => data.json())
      .then((res) => {
        this.me = res;
      });
  },
  methods: {
    confirmdelete(message) {
      if (confirm("Etes vous sure de vouloir supprimer ce post ?")) {
        this.postdelete(message.id)
      }
    },
    publication() {
      const token = this.$store.state.user.token;
      const formdata = new FormData();
      formdata.append("title", this.title);
      formdata.append("content", this.content);
      formdata.append("attachment", this.attachment);

      fetch("http://localhost:3000/api/message/", {
        method: "POST",
        body: formdata,
        headers: {
          Authorization: `Bearer ${token}`,
        },
      })
        .then((res) => {
          return res.json();
        })
        .then((data) => {
          let message = data.message;
          message.User = {
            username: this.$store.state.user.username,
            id: this.$store.state.user.userId,
          };
          console.log(message);
          this.messages.unshift(message);
          this.show = false;
          this.title=''
          this.content=''
          this.attachment=[]
        })
        .catch((error) => console.log(error));
    },
    postdelete(idMessage) {
      const idDelete = this.messages.findIndex(
        (message) => message.id === idMessage
      );
      const token = this.$store.state.user.token;
      fetch(`http://localhost:3000/api/message/${idMessage}`, {
        method: "DELETE",
        headers: {
          Authorization: `Bearer ${token}`,
        },
      })
        .then((res) => {
          return res.json();
        })
        .then((data) => {
          let message = data.message;
          console.log(message);
          this.messages.splice(idDelete, 1);
        });
    },
  },
};
</script>

<style scoped>

.title {
  background-color: rgba(134, 126, 126, 0.778);
}
.headpost {
  display: flex;
  flex-direction: row;
  align-items: center;
  margin: 0 16px 16px 16px;
  justify-content: space-between;
}
.name {
  font-size: 1.3rem;
  margin: 0;
}
.date {
  font-size: 0.75rem;
}
</style>