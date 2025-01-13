<script setup>
import { ref, onMounted, watch } from 'vue';

// Variables réactives
const playlist = ref([]);
const trackSource = ref('url');
const fileInput = ref(null);
const audioPlayer = ref(new Audio());
const currentTrackIndex = ref(-1);
const repeatMode = ref('playlist');
const currentTrackTitle = ref('');
const isPlaying = ref(false);
const currentTime = ref(0);
const duration = ref(0);

// Vérifier si une piste est valide
const checkTrackValidity = (track, index) => {
  const audio = new Audio();
  audio.onerror = () => {
    playlist.value[index].invalid = true;
  };
  audio.onloadedmetadata = () => {
    playlist.value[index].invalid = false;
  };
  audio.src = track.url; // Déclenche l'événement onerror ou onloadedmetadata
};

// Valider toutes les pistes
const validateAllTracks = () => {
  playlist.value.forEach((track, index) => checkTrackValidity(track, index));
};

// Ajouter une piste
const addTrack = (event) => {
  event.preventDefault();
  if (trackSource.value === 'url') {
    const urlInput = event.target.querySelector('input[type="text"]');
    const url = urlInput?.value.trim();
    if (url && url.endsWith('.mp3')) {
      const trackName = url.split('/').pop();
      const newTrack = { name: trackName, url, invalid: false };
      playlist.value.push(newTrack);
      const savedTracks = JSON.parse(localStorage.getItem('urlTracks')) || [];
      savedTracks.push(newTrack);
      localStorage.setItem('urlTracks', JSON.stringify(savedTracks));
      checkTrackValidity(newTrack, playlist.value.length - 1);
      urlInput.value = '';
    } else {
      alert('Veuillez fournir une URL valide se terminant par .mp3.');
    }
  } else if (trackSource.value === 'file') {
    const file = fileInput.value?.files[0];
    if (file) {
      const url = URL.createObjectURL(file);
      const newTrack = { name: file.name, url, invalid: false };
      playlist.value.push(newTrack);
      checkTrackValidity(newTrack, playlist.value.length - 1);
      fileInput.value.value = '';
    } else {
      alert('Veuillez sélectionner un fichier valide.');
    }
  }
};

// Jouer une piste
const playTrack = (index) => {
  if (index >= 0 && index < playlist.value.length && !playlist.value[index].invalid) {
    currentTrackIndex.value = index;
    currentTrackTitle.value = playlist.value[index].name;
    audioPlayer.value.src = playlist.value[index].url;
    audioPlayer.value.play();
    isPlaying.value = true;
  }
};

// Alterner lecture/pause
const togglePlay = () => {
  if (currentTrackIndex.value === -1) {
    if (playlist.value.length > 0) {
      playTrack(0);
    } else {
      alert('La playlist est vide. Ajoutez des pistes pour commencer.');
    }
  } else {
    if (isPlaying.value) {
      audioPlayer.value.pause();
    } else {
      audioPlayer.value.play();
    }
    isPlaying.value = !isPlaying.value;
  }
};

// Supprimer une piste
const deleteTrack = (index) => {
  playlist.value.splice(index, 1);
  if (currentTrackIndex.value === index) {
    audioPlayer.value.pause();
    currentTrackIndex.value = -1;
    currentTrackTitle.value = '';
    isPlaying.value = false;
  }
  const savedTracks = JSON.parse(localStorage.getItem('urlTracks')) || [];
  savedTracks.splice(index, 1);
  localStorage.setItem('urlTracks', JSON.stringify(savedTracks));
};

// Gestion des événements audio
const onTrackEnd = () => {
  if (repeatMode.value === 'track') {
    audioPlayer.value.currentTime = 0;
    audioPlayer.value.play();
  } else if (repeatMode.value === 'playlist') {
    const nextIndex = (currentTrackIndex.value + 1) % playlist.value.length;
    playTrack(nextIndex);
  } else if (repeatMode.value === 'none') {
    const nextIndex = currentTrackIndex.value + 1;
    if (nextIndex < playlist.value.length) {
      playTrack(nextIndex);
    } else {
      currentTrackIndex.value = -1;
      currentTrackTitle.value = '';
      isPlaying.value = false;
    }
  }
};

const updateProgress = () => {
  currentTime.value = audioPlayer.value.currentTime;
  duration.value = audioPlayer.value.duration;
};

const seek = (event) => {
  const progressBar = event.target;
  const clickPosition = event.offsetX / progressBar.offsetWidth;
  audioPlayer.value.currentTime = clickPosition * audioPlayer.value.duration;
};

// Charger les données au montage
onMounted(() => {
  audioPlayer.value.addEventListener('timeupdate', updateProgress);
  audioPlayer.value.addEventListener('ended', onTrackEnd);
  const savedTracks = JSON.parse(localStorage.getItem('urlTracks')) || [];
  playlist.value = savedTracks.map((track) => ({ ...track, invalid: false }));
  validateAllTracks();
});

watch(currentTrackIndex, () => {
  if (currentTrackIndex.value === -1) {
    isPlaying.value = false;
  }
});
</script>

<template>
  <div class="content">
    <h1>Jukebox</h1>
    <section class="player">
      <p>Choisissez une piste pour jouer.</p>
      <div class="mode">
        <fieldset>
          <legend>Mode</legend>
          <label>
            <input type="radio" name="mode" value="playlist" v-model="repeatMode"> Répéter la playlist
          </label>
          <label>
            <input type="radio" name="mode" value="track" v-model="repeatMode"> Répéter la piste
          </label>
          <label>
            <input type="radio" name="mode" value="none" v-model="repeatMode"> Ne pas répéter
          </label>
        </fieldset>
      </div>
      <div v-if="currentTrackTitle" class="current-track">
        <p>Lecture en cours : <strong>{{ currentTrackTitle }}</strong></p>
      </div>
      <div class="custom-audio-controls">
        <button @click="togglePlay">{{ isPlaying ? 'Pause' : 'Lecture' }}</button>
        <div class="progress-bar" @click="seek">
          <div class="progress" :style="{ width: `${(currentTime / duration) * 100}%` }"></div>
        </div>
        <span>{{ Math.floor(currentTime / 60) }}:{{ (currentTime % 60).toFixed(0).padStart(2, '0') }}</span>
      </div>
    </section>

    <section class="playlist">
      <h2>Playlist</h2>
      <ul>
        <li v-for="(track, index) in playlist" :key="index" :class="{ playing: currentTrackIndex === index, invalid: track.invalid }">
          {{ track.name }}
          <div class="buttons">
            <button @click="playTrack(index)" :disabled="track.invalid">Play</button>
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
        <select id="track-source" name="track-source" v-model="trackSource">
          <option value="url">Par URL</option>
          <option value="file">Par chargement de fichier</option>
        </select>
        <input type="text" placeholder="Fournir l'URL (terminant par .mp3)" v-if="trackSource === 'url'">
        <input type="file" ref="fileInput" v-if="trackSource === 'file'">
        <button type="submit">{{ trackSource === 'url' ? 'Ajouter par URL' : 'Ajouter par fichier' }}</button>
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
  list-style: none;
  padding: 0;
}

li {
  padding: 5px 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-radius: 5px;
  margin: 5px 0;
}

.playing {
  font-weight: bold;
  color: #002142;
}

.invalid {
  text-decoration: line-through;
  color: red;
}

.buttons {
  display: flex;
  gap: 10px;
}

.progress-bar {
  flex: 1;
  height: 8px;
  background-color: #ddd;
  border-radius: 4px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.progress {
  height: 100%;
  background-color: #007BFF;
  width: 0;
}

.mode fieldset {
  border: 2px solid #007BFF;
  border-radius: 8px;
  padding: 10px 15px;
  margin-top: 10px;
  display: inline-block;
}

.mode legend {
  padding: 0 5px;
  font-size: 14px;
  font-weight: bold;
  color: #007BFF;
}
</style>
