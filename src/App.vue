<script setup lang="ts">
import { ref } from "vue";
import { open } from "@tauri-apps/api/dialog";
import { readTextFile, writeTextFile } from "@tauri-apps/api/fs";
import { readDir } from '@tauri-apps/api/fs';

const folderPath = ref("");
const processStatus = ref("");
const isProcessing = ref(false);

const selectFolder = async () => {
  try {
    const selectedPath = await open({
      directory: true,
      multiple: false,
    });

    if (!selectedPath) return;
    folderPath.value = selectedPath as string;
    processStatus.value = "Folder selected. Click Process to start.";
  } catch (err) {
    console.error(err);
    processStatus.value = "Error selecting folder: " + err;
  }
};

const processFiles = async () => {
  if (!folderPath.value) {
    processStatus.value = "Please select a folder first!";
    return;
  }

  try {
    isProcessing.value = true;
    processStatus.value = "Processing files...";

    const files = await readDir(folderPath.value);
    let processedCount = 0;

    for (const file of files) {
      if (file.name?.endsWith('.txt')) {
        console.log(file.path)
        const content = await readTextFile(file.path);
        const lines = content.split('\n');
        // replace all the lines that contain the pattern with an empty string
        const filteredLines = lines.map(line => {
          return line.replace(/\d{2}:\d{2}:\d{2}:\d{2}\s+\d{2}:\d{2}:\d{2}:\d{2}/, '');
        });
        await writeTextFile(file.path, filteredLines.join('\n'));
        processedCount++;
      }
    }

    processStatus.value = `Processing completed! Processed ${processedCount} files.`;
  } catch (err) {
    console.error(err);
    processStatus.value = "Error processing files: " + err;
  } finally {
    isProcessing.value = false;
  }
};
</script>

<template>
  <div class="container">
    <h4>File Processor</h4>

    <div class="actions">
      <button @click="selectFolder" :disabled="isProcessing">
        Select Folder
      </button>
      <button
        @click="processFiles"
        :disabled="!folderPath || isProcessing"
        class="process-btn"
      >
        {{ isProcessing ? 'Processing...' : 'Process Files' }}
      </button>
    </div>

    <div class="status">
      <p v-if="folderPath" class="folder-path">
        Selected folder: {{ folderPath }}
      </p>
      <p v-if="processStatus" class="status-message">
        {{ processStatus }}
      </p>
    </div>
  </div>
</template>

<style>
html, body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}

.container {
  padding: 1rem;
  height: 380px;
  margin: 0;
  background-color: #ffffff;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

h4 {
  color: #1a2942;
  margin: 0 0 1rem 0;
  font-size: 1.2rem;
  font-weight: 600;
  text-align: center;
}

.actions {
  display: flex;
  gap: 0.8rem;
  justify-content: center;
  margin-bottom: 1rem;
}

button {
  padding: 0.4rem 0.8rem;
  border-radius: 6px;
  font-size: 0.85rem;
  min-width: 100px;
  border: 1px solid #e5e7eb;
}

.status {
  flex: 1;
  overflow: hidden;
}

.folder-path {
  padding: 0.6rem;
  font-size: 0.85rem;
  margin: 0;
  word-break: break-all;
  background-color: #f9fafb;
  border-radius: 6px;
  overflow: hidden;
  text-overflow: ellipsis;
}

.status-message {
  padding: 0.6rem;
  font-size: 0.85rem;
  margin-top: 0.6rem;
  background-color: #f9fafb;
  border-radius: 6px;
}
</style>
