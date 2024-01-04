<template>
  <div class="container">
    <h1>Bienvenue</h1>

    <!-- Debut -->
    <div  v-if="!Debut"  class="SectionDebut">
      <label  for= "name"> Entrer votre Nom : </label>
      <input  v-model= "joueur"  type ="text"  id="name" /> <!-- pour recuperer le nom du joueur -->
      <br />
      <label  for ="theme"> Choisissez un theme:</label>
      <select  v-model= "selectionTheme"  id="theme"> <!-- Pour recuperer le theme choisi -->
        <option  v-for= "(theme, index)  in  themes"  :key="index"  :value="index"> {{ theme }} </option> <!-- on affiche les themes -->
      </select>
      <br />
      <button  @click="commencer"> Commencer </button>
    </div>

    <!-- Quiz -->
    <div  v-else  class= "SectionQuiz">
      <h2  v-if="questionActuel">{{ questionActuel.text }}</h2>  <!-- on pose la question  -->
      <!-- pour la question actuelle on ressort les options de reponse  -->
      <ul v-if="questionActuel">
        <li v-for="(option, index) in questionActuel.options" :key="index">
          <button @click="accepterReponse(option, questionActuel.audio)">{{ option }}</button> <!-- on accepte la reponse avec audio a lappuie -->
        </li>
      </ul>
      <p>Score: {{ score }}</p>
      <button @click="questionSuivante">Next Question</button>
      <audio ref="correctAudio" :src="correctAnswerAudio"></audio>
    </div>

    <!-- Fin -->
    <div v-if="fin" class="SectionFinQuiz">
      <h2>Quiz Finished!</h2>
      <h2>Name: {{ utilisateurActuel.name }}</h2>
      <h2>Theme: {{ themes[utilisateurActuel.selectionTheme] }}</h2>
      <h2>Score: {{ score }}</h2>
      <button @click="EnregistrerResultat" type="button">Save Results</button>
      <button @click="prochainUser" type="button">Show User</button>
      <button @click="reload" type="button">Another part</button>
    </div>

    <!-- Resultats -->
    <div v-if="MontrerResultats" class="SectionResultat">
      <h3>Quiz Results</h3>
      <button @click="trier">Sort Results</button>
      <table>
        <thead>
        <tr>
          <th>Name</th>
          <th>Theme</th>
          <th>Score</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="(result, index) in resultat" :key="index">
          <td>{{ result.name }}</td>
          <td>{{ result.theme }}</td>
          <td>{{ result.score }}</td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //declaration
      joueur: '',
      themes: ["Histoire", "Géographie","football","basketball"],
      selectionTheme: null,
      Debut: false,
      fin: false,
      indexQuestion: 0,
      backgroundMusic:null,
      score: 0,
      utilisateur: [],
      utilisateurActuel: {}, // Initialize utilisateurActuel as an empty object
      MontrerResultats: false,
      jouerMusique:false,
      questions: [
        { text: "Quelle est la capitale de la France ?", options: ["Paris", "Paris", "Londres", "Madrid"], answer: "Paris", theme: "Géographie", audio: require('./only.mp3') },
        { text: "Quelle est la capitale de la Guinée ?", options: ["Dakar", "Bamako", "Londres","Conakry"], answer: "Conakry", theme: "Géographie", audio: require('./only.mp3') },
        { text: "Quelle est la plus grande rivière du monde ?", options: ["Amazone","Nil", "Mississippi",  "Yangtsé"], answer: "Amazone", theme: "Géographie", audio: require('./only.mp3') },
        { text: "Quel pays est le plus grand en superficie ?", options: ["États-Unis", "Chine", "Canada", "Russie"], answer: "Russie", theme: "Géographie", audio: require('./only.mp3') },
        { text: "Quel est le plus haut sommet du monde ?", options: ["Mont Everest", "K2", "Kangchenjunga", "Lhotse"], answer: "Mont Everest", theme: "Géographie", audio: require('./only.mp3') },
        { text: "Quel désert est le plus grand au monde ?", options: ["Désert du Sahara", "Désert d'Arabie", "Désert du Gobi", "Désert du Kalahari"], answer: "Désert de l'Antarctique", theme: "Géographie", audio: require('./only.mp3') },
        { text: "Quelle est la capitale du Japon ?", options: ["Tokyo", "Séoul", "Pékin", "Bangkok"], answer: "Tokyo", theme: "Géographie", audio: require('./only.mp3') },
        // Theme: Histoire
        { text: "En quelle année a eu lieu la Révolution française ?", options: ["1789", "1804", "1850", "1900"], answer: "1789", theme: "Histoire", audio: require('./only.mp3') },
        { text: "Quelle guerre a été appelée 'la Grande Guerre' ?", options: ["Guerre froide", "Guerre de Sécession", "Première Guerre mondiale", "Seconde Guerre mondiale"], answer: "Première Guerre mondiale", theme: "Histoire", audio: require('./only.mp3') },
        { text: "Qui était le président des États-Unis pendant la guerre civile américaine ?", options: ["Abraham Lincoln", "George Washington", "Thomas Jefferson", "John F. Kennedy"], answer: "Abraham Lincoln", theme: "Histoire", audio: require('./only.mp3') },
        { text: "Quel empereur romain a construit le Colisée ?", options: ["Auguste", "Néron", "Trajan", "Vespasien"], answer: "Vespasien", theme: "Histoire", audio: require('./only.mp3') },
        { text: "Quel événement a marqué le début de la Renaissance en Europe ?", options: ["Chute de Constantinople", "Découverte de l'Amérique", "Invention de l'imprimerie", "Bataille de Hastings"], answer: "Chute de Constantinople", theme: "Histoire", audio: require('./only.mp3') },
        { text: "Qui était le roi de France pendant la Révolution française ?", options: ["Louis XIV", "Louis XV", "Louis XVI", "Napoléon Bonaparte"], answer: "Louis XVI", theme: "Histoire", audio: require('./only.mp3') },
        { text: "Quel explorateur a réalisé le premier tour du monde en 1522 ?", options: ["Marco Polo", "Christophe Colomb", "Ferdinand Magellan", "Vasco de Gama"], answer: "Ferdinand Magellan", theme: "Histoire", audio: require('./only.mp3') },
        // Theme: Sports (Football)
        { text: "Qui a remporté la Coupe du Monde de football en 2018 ?", options: ["Brésil", "Allemagne", "France", "Espagne"], answer: "France", theme: "football", audio: require('./only.mp3') },
        { text: "Quel joueur a remporté le Ballon d'Or en 2021 ?", options: ["Lionel Messi", "Cristiano Ronaldo", "Robert Lewandowski", "Mohamed Salah"], answer: "Lionel Messi", theme: "football", audio: require('./only.mp3') },
        { text: "Dans quel pays se trouve le stade de football Maracanã ?", options: ["Brésil", "Argentine", "Uruguay", "Chili"], answer: "Brésil", theme: "Sports", audio: require('./only.mp3') },
        { text: "Quel club a remporté le plus grand nombre de Ligue des champions de l'UEFA ?", options: ["Real Madrid", "FC Barcelone", "Bayern Munich", "Liverpool"], answer: "Real Madrid", theme: "football", audio: require('./only.mp3') },
        { text: "Qui est le meilleur buteur de tous les temps en Coupe du Monde ?", options: ["Pelé", "Miroslav Klose", "Diego Maradona", "Ronaldo Nazário"], answer: "Miroslav Klose", theme: "football", audio: require('./only.mp3') },
        { text: "Quel pays a remporté le plus grand nombre de Coupes d'Afrique des Nations ?", options: ["Cameroun", "Nigeria", "Égypte", "Ghana"], answer: "Égypte", theme: "football", audio: require('./only.mp3') },
        { text: "Quel joueur a été surnommé 'La Pulga' ?", options: ["Neymar", "Lionel Messi", "Cristiano Ronaldo", "Kylian Mbappé"], answer: "Lionel Messi", theme: "football", audio: require('./only.mp3') },

// Theme: Sports (Basketball)
        { text: "Quelle équipe a remporté le championnat de la NBA en 2021 ?", options: ["Los Angeles Lakers", "Miami Heat", "Phoenix Suns", "Milwaukee Bucks"], answer: "Milwaukee Bucks", theme: "basketball", audio: require('./only.mp3') },
        { text: "Qui est le meilleur marqueur de tous les temps en NBA ?", options: ["LeBron James", "Kobe Bryant", "Kareem Abdul-Jabbar", "Michael Jordan"], answer: "Kareem Abdul-Jabbar", theme: "basketball", audio: require('./only.mp3') },
        { text: "Quel joueur de basketball est souvent appelé 'The Greek Freak' ?", options: ["Luka Dončić", "Giannis Antetokounmpo", "Joel Embiid", "Anthony Davis"], answer: "Giannis Antetokounmpo", theme: "basketball", audio: require('./only.mp3') },
        { text: "Combien de points vaut un lancer franc au basketball ?", options: ["1 point", "2 points", "3 points", "4 points"], answer: "1 point", theme: "basketball", audio: require('./only.mp3') },
        { text: "Quelle équipe a remporté le plus grand nombre de championnats de la NBA ?", options: ["Los Angeles Lakers", "Boston Celtics", "Chicago Bulls", "Golden State Warriors"], answer: "Boston Celtics", theme: "basketball", audio: require('./only.mp3') },
        { text: "Qui a remporté le titre de MVP de la NBA en 2020 ?", options: ["Giannis Antetokounmpo", "LeBron James", "James Harden", "Anthony Davis"], answer: "Giannis Antetokounmpo", theme: "basketball", audio: require('./only.mp3') },
        { text: "Quel joueur a été surnommé 'The Black Mamba' ?", options: ["Kevin Durant", "Kawhi Leonard", "Dwyane Wade", "Kobe Bryant"], answer: "Kobe Bryant", theme: "basketball", audio: require('./only.mp3') },

        // Ajoute d'autres questions ici
      ],
    };
  },
  computed: {
    filtrerQuestion() {// ici on ressort les questions dun meme theme
      if (this.selectionTheme !== null) {
        const selectionTheme = this.themes[this.selectionTheme];
        return this.questions.filter(question => question.theme === selectionTheme);
      }
      return [];
    },
    questionActuel() { //on se pose sur la question actuelle en utilisant la longueur
      if (this.filtrerQuestion.length > 0) {
        return this.filtrerQuestion[this.indexQuestion];
      }
      return null;
    },
    resultat() {
      const results = [];
      for (let i = 0; i < this.utilisateur.length; i++) {
        const user = this.utilisateur[i];
        results.push({
          name: user.name,
          theme: this.themes[user.selectionTheme],
          score: user.score,
        });
      }
      return results;
    },
  },
  methods: {
// methode pour faire apparaitre les questions dun meme theme au hasard
    questionAleatoire() {
      for (let i = this.filtrerQuestion.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [this.filtrerQuestion[i], this.filtrerQuestion[j]] = [this.filtrerQuestion[j], this.filtrerQuestion[i]];
      }
    },

    commencer() {
      if (this.joueur.trim() === '') {
        alert('Please enter your name before starting the quiz.');
        return;
      }

      this.Debut = true;
      this.utilisateurActuel = {
        name: this.joueur,
        selectionTheme: this.selectionTheme,
        score: 0,
      };
      this.questionAleatoire();
      if(!this.jouerMusique){
        this.backgroundMusic=new Audio(require('./Jim Yosef.mp3'));
        this.backgroundMusic.play();
        this.backgroundMusic.loop=true;
        this.jouerMusique=true;
      }


    },
    accepterReponse(selectedOption, audio) {
      const questionActuel = this.questionActuel;
      if (questionActuel && selectedOption === questionActuel.answer) {
        this.score++;
        // Utilisez setTimeout pour retarder l'appel de Musique
        setTimeout(() => {
          this.Musique(audio);
        }, 0);
      } else {
        alert("Incorrect.");
      }
      this.questionSuivante();
    },

    Musique(audio){
      this.$refs.correctAudio.src=audio;
      this.$refs.correctAudio.play();
    },
    questionSuivante() {
      this.indexQuestion++;
      if (this.indexQuestion >= this.filtrerQuestion.length) {
        this.fin = true;// on sassure de terminer le quiz
      }
    },
    EnregistrerResultat() {// nous sommes dans l'optique ou le meme joueur ne joue qu'une fois pour  un quiz complet il ne peut rejouer que si
      //une nouvelle partie est lancer
      if (!this.utilisateur.some(user => user.name === this.utilisateurActuel.name)) {
      this.utilisateurActuel.score = this.score;
      this.utilisateur.push(this.utilisateurActuel);
    }
    },
    stopMusic() {
      if (this.backgroundMusic) {
        this.backgroundMusic.pause();
        this.backgroundMusic.currentTime = 0;
        this.jouerMusique=false;

      }
    },
    reload(){
      this.recommencer();
    },
    recommencer() {
      this.Debut = false;
      this.fin = false;
      this.indexQuestion = 0;
      this.score = 0;
      this.joueur = '';
      this.selectionTheme = null;
      this.utilisateurActuel = {}; // Reset utilisateurActuel to an empty object
    },

    prochainUser() {
      if (this.utilisateur.length >=0) {
        this.MontrerResultats = true;
        this.stopMusic();
      } else {
        this.MontrerResultats = false;
        this.commencer();
      }
    },

    trier() { // si la reponse est negatif b est plus grand ond on le place on haut
      this.utilisateur.sort((a, b) => b.score - a.score);
    },
    beforeDestroy() {
      if (this.backgroundMusic) {
        this.backgroundMusic.pause();
        this.backgroundMusic.currentTime = 0;
      }
    },
  },
};
</script>
<style scoped>
.container {

  text-align: center;
   margin: 0;
   background-image: url('../assets/img.JPG');
   background-size: cover;
   background-repeat: no-repeat;
   background-position: center;
   justify-content: center;
   min-height: 100vh;
   height:100%;

}
ul{
  list-style-type: none;
}
p{color:red;margin-bottom: 20px;padding: 8px 12px;}
h1{color: red;font-weight: bold;padding: 8px 12px;}
h2{color: red;font-weight: bold;padding: 8px 12px;}
h3{color: aliceblue;margin-bottom: 20px;}

.SectionDebut {
  margin-bottom: 20px;
   color: red;
    font-weight: bold;
}

.SectionQuiz {
    margin-bottom: 20px;
   color: red;
}

.SectionFinQuiz {
   margin-bottom:  20px;
  color:  red;
}

.SectionResultat {
  margin-bottom: 20px;
  color:  red;
}

button {
  cursor:  pointer;
  padding:  8px 12px;
  margin: 5px;
}

table {
  width:  100%;
  border-collapse:  collapse;
  margin-top:  10px;
}

th, td {
  border:  1px solid gold;
  padding:  8px ;
  text-align:  left;
}
</style>
