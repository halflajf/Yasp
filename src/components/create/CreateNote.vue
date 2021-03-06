<template>
  <v-container>
    <h1 align="center">
      Create new note
      <v-icon color="black">event_note</v-icon>
    </h1>
    <br />
    <br />
    <br />
    <v-row>
      <v-col>
        <v-form ref="form">
          <v-text-field
            filled
            v-model="note.title"
            label="Note title (required)"
            :color="appColor"
            :rules="[v => !!v || 'Title is required']"
          ></v-text-field>
          <v-textarea
            filled
            dense
            v-model="note.content"
            name="note-content"
            label="Note content (required)"
            rows="5"
            :color="appColor"
            :rules="[v => !!v || 'Content is required']"
          ></v-textarea>
          <v-flex xs12>
            <v-combobox
              filled
              multiple
              v-model="note.tags"
              label="Tags (optional)"
              append-icon="label"
              chips
              deletable-chips
              :search-input.sync="search"
              @keyup.tab="updateTags"
              @paste="updateTags"
              :color="appColor"
              hint="Press enter after typing each tag to add it to list"
            ></v-combobox>
          </v-flex>
          <v-layout align-end justify-end>
            <v-btn
              :disabled="note.title.length == 0 || note.content.length == 0"
              :color="appColor"
              @click="addNote()"
              >Add note</v-btn
            >
          </v-layout>
        </v-form>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import Vue from "vue";
import { mapGetters, mapActions } from "vuex";
import { v4 as uuidv4 } from "uuid";
import { JsonBinApi } from "../../JsonBinApi";

export default Vue.extend({
  name: "CreateNote",

  data() {
    return {
      note: {
        title: "",
        content: "",
        tags: Array<string>(),
        jsonbinId: ""
      },
      search: ""
    };
  },
  computed: {
    ...mapGetters(["allTags", "appColor"])
  },
  methods: {
    ...mapActions(["commitNote", "commitEditNote"]),
    updateTags() {
      this.$nextTick(() => {
        this.note.tags.push(...this.search.split(","));
        this.$nextTick(() => {
          this.search = "";
        });
      });
    },
    async addNote() {
      const date = new Date();
      const note = {
        ...this.note,
        id: uuidv4(),
        whenCreated: new Date(date.getTime() - date.getTimezoneOffset() * 60000)
          .toISOString()
          .slice(0, -5)
          .replace("T", ", "),
        whenEdited: new Date(date.getTime() - date.getTimezoneOffset() * 60000)
          .toISOString()
          .slice(0, -5)
          .replace("T", ", ")
      };
      this.commitNote(note);
      //handle result
      const result = await JsonBinApi.addNote(note);
      this.commitEditNote({ ...note, jsonbinId: result.id });
      await JsonBinApi.updateNote(result.id, {
        ...note,
        jsonbinId: result.id
      });
      this.note.title = "";
      this.note.content = "";
      this.note.tags = [];
      this.search = "";
      // eslint-disable-next-line
      (this as any).$refs.form.resetValidation();
      this.$router.push({ name: "Home", query: { createdTitle: note.title } });
    }
  }
});
</script>
