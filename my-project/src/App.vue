<script setup>
import { ref } from 'vue';

const trackSource = ref('url'); // Valeur initiale : URL
</script>

<template>
  <div class="content">
    <h1>Jukebox</h1>
    <section class="player">
      <p>Choisissez une piste pour jouer.</p>
      <div class="mode">
        <label>
          <input type="radio" name="mode" checked>
          Répéter la playlist
        </label>
        <label>
          <input type="radio" name="mode">
          Répéter la piste
        </label>
        <label>
          <input type="radio" name="mode">
          Ne pas répétez
        </label>
      </div>
    </section>

    <section class="playlist">
      <h2>Playlist</h2>
      <p>Ajoutez une nouvelle piste à la liste de lecture.</p>
    </section>

    <section class="new-track">
      <h2>Nouvelle piste</h2>
      <form>
        <label for="track-source">Ajouter une piste :</label>
        <select 
          id="track-source" 
          name="track-source" 
          v-model="trackSource"
        >
          <option value="url">Par URL</option>
          <option value="file">Par chargement de fichier</option>
        </select>
        <input 
          type="text" 
          placeholder="Fournir l'URL" 
          v-if="trackSource === 'url'"
        >
        <input 
          type="file" 
          v-if="trackSource === 'file'"
        >
        <button type="submit">
          {{ trackSource === 'url' ? 'Ajouter par URL' : 'Ajouter par fichier' }}
        </button>
      </form>
    </section>
  </div>
</template>

<style scoped>
  body {
    font-family: Arial, sans-serif;
    line-height: 1.6;
    margin: 0;
    padding: 0;
    background-color: #f9f9f9;
    color: #333;
  }

  h1, h2 {
      text-align: center;
      color: #222;
  }

  section {
      margin-bottom: 20px;
  }

  .mode label {
      display: inline-block;
      margin-right: 15px;
      font-size: 14px;
      cursor: pointer;
  }

  input[type="radio"] {
      margin-right: 5px;
  }

  form {
      display: flex;
      gap: 10px;
      align-items: center;
      flex-wrap: wrap;
  }

  input[type="text"], input[type="file"] {
      flex: 1;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px;
  }

  button {
      padding: 8px 12px;
      background-color: #007BFF;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
  }

  button:hover {
      background-color: #0056b3;
  }
</style>
