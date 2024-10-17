<template>
  <div class="flex w-screen h-screen text-gray-700">
    <div class="flex flex-col flex-shrink-0 w-64 border-r border-gray-300 bg-gray-100">
      <!-- Sidebar -->
    </div>
    <div class="flex flex-col flex-grow">
      <!-- Main Content -->
      <div class="flex flex-col flex-grow overflow-auto">
        <editor-content :editor="editor" />
      </div>
      <div class="h-16 bg-gray-100 border-t border-gray-300 flex items-center justify-end p-4">
        <button @click="saveNote" class="px-4 py-2 bg-gray-500 text-white rounded hover:bg-black focus:outline-none">Save Note</button>
      </div>
    </div>
  </div>
</template>


<script>
import { Editor, EditorContent } from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'

export default{
  name: 'App',
  components: {
    EditorContent
  },
  data() {
    return{
      editor:null,
      database:null,
    }
  },
  async created(){
    this.database = await this.getDatabase();

  },
  mounted(){
    this.editor=new Editor({
      content: '',
      extensions: [
        StarterKit
      ],
      editorProps:{
        attributes:{
          class: "prose my-6 mx-auto focus:outline-none"
        }
      }
    });
  },
  beforeUnmount(){
    this.editor.destroy();
  },
  methods:{
    async getDatabase(){
      return new Promise((resolve, reject) => {
        let db = window.indexedDB.open("notes", 2);

        db.onerror = e => {
          reject('Error opening the database.')
        };
        db.onsuccess = e => {
          console.log('db.onsuccess', e)
          resolve(e.target.result);
        };
        db.onupgradeneeded = e => {
          console.log('db.onupgraded', e)
          e.target.result.deleteObjectStore("notes");
          e.target.result.createObjectStore("notes", { keyPath: "created" });
        };
      });
    },
    async saveNote(){
      return new Promise((resolve, reject) => {
        let transaction = this.database.transaction('notes', 'readwrite')
        transaction.oncomplete = e => {
          resolve();
        }
        let now =new Date();
        transaction.objectStore('notes').add({
          content: this.editor.getHTML(),
          created: now.getTime()
        });
      });
    }
  }
}

</script>



<style></style>
