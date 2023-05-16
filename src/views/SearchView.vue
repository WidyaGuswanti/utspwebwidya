<template>
  <body>
  <!-- NavBar -->
  <nav class="navbar navbar-expand-lg navbar-dark bg-dark shadow fixed-top">
    <div class="container">
      <a class="navbar-brand" href="#">Al-Qur'an</a>
      <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
      </button>

    </div>
  </nav>
  <!-- Akhir NavBar -->
  </body>

  <section class="search">
    <input
        type="number"
        v-model="inputnomor"
        class="input"
        placeholder="Masukkan nomor surah"
    />
    <button @click="lihat" class="btn">Cari</button>
  </section>

  <section class="surah">
    <div class="judull">
      <h1 v-if="namaSurah" class="judul">{{ namaSurah.name_simple }}</h1>
    </div>

    <div class="suara">
      <p v-if="audio">
        <audio controls class="suaraa">
          <source :src="audio.audio_url" type="audio/mpeg" />
          Your browser does not support the audio element.
        </audio>
      </p>
    </div>

  </section>

  <section class="hasil">
    <div class="hasill">
      <ul class="lista">
        <li class="ayat" v-for=" (ayat) in ayah ">
          {{ayat.text_uthmani}} {{ayat.text}}
        </li>
      </ul>
    </div>
  </section>
</template>
<script>
import axios from "axios";
import {ref} from "vue";

export default {
  data() {
    return {
      ayah: [ref],
      audio: null,
      namaSurah: null,
      inputnomor: "",
    };
  },

  methods: {
    async lihat() {
      let nomor = this.inputnomor;
      let ayat = `https://api.quran.com/api/v4/quran/verses/uthmani?chapter_number=${nomor}`;
      let arti = "https://api.quran.com/api/v4/quran/translations/33?chapter_number=" +
          nomor;

      let judul = "https://api.quran.com/api/v4/chapters?language=en";
      let suara =
          "https://api.quran.com/api/v4/chapter_recitations/2?language=en";

      if (nomor <= 0 || nomor > 114) {
        alert("masukkan nomor dengan benar");
      } else {
        const reqAyat = axios.get(ayat);
        const reqArti = axios.get(arti);
        const reqJudul = axios.get(judul);
        const reqSuara = axios.get(suara);

        axios.all([reqAyat, reqArti, reqJudul, reqSuara]).then(
            axios.spread((...response) => {
              const responseAyat = response[0];
              const responseArti = response[1];
              const responseJudul = response[2];
              const responseSuara = response[3];

              const a = responseAyat.data.verses;
              const b = responseArti.data.translations;

              const gabung = (a, b) => {
                const res = [];

                for (let i = 0; i < a.length + b.length; i++) {
                  if (i % 2 === 0) {
                    res.push(a[i / 2]);
                  } else {
                    res.push(b[(i - 1) / 2]);
                  }
                }
                return res;
              };

              this.ayah= gabung(a, b);
              this.audio =
                  responseSuara.data.audio_files[nomor - 1];
              this.namaSurah =
                  responseJudul.data.chapters[nomor - 1];
            })
        );
      }
    },
  },
};
</script>
<style>
.search {
  display: flex;
  margin: 50px 0 0 0;
  flex-direction: row-reverse;
  justify-content: center;
}

.input {
  height: 30px;
  border: 1px solid #323232;
  color: rgb(0, 0, 0);
}

.input:hover {
  border: 1px solid #957c6d;
}

.btn {
  background-color: rgb(27, 176, 23);
  border: 1px solid #323232;
  height: 30px;
  font-size: 15px;
  color: black;
  border-radius: 5%;
}

.btn:hover {
  color: #1bb017;
}

.suaraa {
  width: 70%;
  height: 30px;
}

.suara {
  margin: 50px 0;
  text-align: center;
}

.judul {
  text-align: center;
  font-size: 50px;
  color: #323232;
  margin: 60px 0 0 30px;
}

.bismillah {
  text-align: center;
  font-size: 30px;
  color: #323232;
}

.ayat {
  color: #323232;
  list-style: none;
  margin: 0 100px 50px 200px;
}

.ayat:nth-child(odd) {
  text-align: center;
  font-size: 20px;
}
.ayat:nth-child(even) {
  text-align: center;
  font-size: 15px;
  color: #323232;
}

@media screen and (max-width: 300px) {
  .ayat:nth-child(odd) {
    font-size: 10px;
    margin-bottom: 20px;
    margin-right: 10px;
  }
  .ayat:nth-child(even) {
    font-size: 10px;
    margin: 20px;
  }
  .ayat {
    margin: 0;
  }
}
</style>