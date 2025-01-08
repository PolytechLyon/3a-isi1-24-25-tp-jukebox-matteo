<script setup>
import { ref } from 'vue';

// Liste réactive pour la playlist
const playlist = ref([]);

// Gestion de la source du fichier
const trackSource = ref('url'); // Valeur par défaut : URL
const fileInput = ref(null); // Référence au champ fichier

// Référence pour le lecteur audio
const audioPlayer = ref(null); // Référence pour le lecteur audio HTML5
const currentTrackIndex = ref(-1); // Index de la piste actuellement en cours
const repeatMode = ref('playlist'); // Mode de répétition par défaut

// Ajouter une piste à la playlist
const addTrack = (event) => {
  event.preventDefault();

  if (trackSource.value === 'file') {
    const file = fileInput.value?.files[0];
    if (file) {
      const url = URL.createObjectURL(file); // Crée un URL temporaire pour le fichier
      playlist.value.push({ name: file.name, url }); // Ajoute la piste avec son URL
      fileInput.value.value = ''; // Réinitialise le champ fichier
    } else {
      alert("Veuillez sélectionner un fichier valide.");
    }
  }
};

// Jouer une piste sélectionnée
const playTrack = (index) => {
  if (index >= 0 && index < playlist.value.length) {
    currentTrackIndex.value = index;
    if (audioPlayer.value) {
      audioPlayer.value.src = playlist.value[index].url; // Définit la source
      audioPlayer.value.play(); // Joue la piste
    }
  }
};

// Supprimer une piste de la playlist
const deleteTrack = (index) => {
  playlist.value.splice(index, 1);
  if (currentTrackIndex.value === index) {
    audioPlayer.value.pause();
    currentTrackIndex.value = -1; // Réinitialise la piste en cours si supprimée
  }
};

// Gérer la fin de la lecture
const onTrackEnd = () => {
  if (repeatMode.value === 'track') {
    // Répéter la piste actuelle
    audioPlayer.value.currentTime = 0;
    audioPlayer.value.play();
  } else if (repeatMode.value === 'playlist') {
    // Passer à la piste suivante ou redémarrer la playlist
    const nextIndex = (currentTrackIndex.value + 1) % playlist.value.length;
    playTrack(nextIndex);
  } else if (repeatMode.value === 'none') {
    // Passer à la piste suivante sans boucle
    const nextIndex = currentTrackIndex.value + 1;
    if (nextIndex < playlist.value.length) {
      playTrack(nextIndex);
    } else {
      // Arrêter la lecture si c'était la dernière piste
      currentTrackIndex.value = -1;
    }
  }
};
</script>

<template>
  <div class="content">
    <h1>Jukebox</h1>
    <section class="player">
      <p>Choisissez une piste pour jouer.</p>
      <div class="mode">
        <label>
          <input 
            type="radio" 
            name="mode" 
            value="playlist" 
            v-model="repeatMode"
          >
          Répéter la playlist
        </label>
        <label>
          <input 
            type="radio" 
            name="mode" 
            value="track" 
            v-model="repeatMode"
          >
          Répéter la piste
        </label>
        <label>
          <input 
            type="radio" 
            name="mode" 
            value="none" 
            v-model="repeatMode"
          >
          Ne pas répétez
        </label>
      </div>
      <audio ref="audioPlayer" controls @ended="onTrackEnd" style="width: 100%; margin-top: 20px;"></audio>
    </section>

    <section class="playlist">
      <h2>Playlist</h2>
      <ul>
        <!-- Affichage des pistes dans la playlist avec boutons "Play" et "Supprimer" -->
        <li v-for="(track, index) in playlist" :key="index">
          {{ track.name }}
          <div class="buttons">
            <button @click="playTrack(index)">Play</button>
            <button @click="deleteTrack(index)">Supprimer</button>
          </div>
        </li>
      </ul>
      <p v-if="playlist.length === 0">Ajoutez une nouvelle piste à la liste de lecture.</p>
    </section>

    <section class="new-track">
      <h2>Nouvelle piste</h2>
      <form @submit="addTrack">
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
          ref="fileInput" 
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

  ul {
    list-style: disc;
    padding-left: 20px;
  }

  li {
    padding: 5px 0;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .buttons {
    display: flex;
    gap: 10px;
  }
</style>
