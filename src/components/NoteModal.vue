<template>
  <v-dialog v-model="showNoteDialog" persistent max-width="500">
    <v-card>
      <div class="pa-2">
        <v-text-field
          label="Title"
          single-line
          flat
          solo
          :value="selectedNote.title"
          @input="updateTitle"
        ></v-text-field>
        <v-textarea
          auto-grow
          flat
          solo
          label="Take a note..."
          single-line
          :value="selectedNote.content"
          @input="updateContent"
        ></v-textarea>
      </div>
      <v-card-actions>
        <v-btn icon @click="deleteNote" :loading="deleteLoading">
          <v-icon>mdi-delete-outline</v-icon>
        </v-btn>
        <v-spacer></v-spacer>
        <v-btn text @click="setShowNoteDialog(false)">Close</v-btn>
        <v-btn text @click="saveNote" :loading="loading">Save</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { Note, Snackbar, EditNoteField } from '@/store/models';
import { SnackbarColorTypes } from '@/store/enums';
import { namespace } from 'vuex-class';

const notesModule = namespace('notes');
const globalModule = namespace('global');

@Component
export default class NoteModal extends Vue {
  loading = false;
  deleteLoading = false;

  @globalModule.State
  showNoteDialog!: boolean;

  @globalModule.State
  selectedNote!: Note;

  @globalModule.Mutation
  setShowNoteDialog!: (payload: boolean) => void;

  @globalModule.Mutation
  showSnackbar!: (snackbar: Snackbar) => void;

  @globalModule.Mutation
  setNoteField!: (field: EditNoteField) => void;

  @notesModule.Action
  updateNote!: (note: Note) => Promise<Note>;

  @notesModule.Action
  removeNoteAsync!: (id: string) => Promise<string>;

  async saveNote(): Promise<void> {
    this.loading = true;
    await this.updateNote(this.selectedNote);
    this.loading = false;
    this.showSnackbar({
      open: true,
      text: 'Note updated',
      color: SnackbarColorTypes.Success,
    });
    this.setShowNoteDialog(false);
  }

  async deleteNote(): Promise<void> {
    this.deleteLoading = true;
    await this.removeNoteAsync(this.selectedNote.id);
    this.deleteLoading = false;
    this.showSnackbar({
      open: true,
      text: 'Note removed',
      color: SnackbarColorTypes.Success,
    });
    this.setShowNoteDialog(false);
  }

  updateTitle(value: string): void {
    this.setNoteField({
      name: 'title',
      value,
    });
  }

  updateContent(value: string): void {
    this.setNoteField({
      name: 'content',
      value,
    });
  }
}
</script>
