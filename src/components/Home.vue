<template>
  <div>
    <!-- The assign seats section -->
    <div class="row">

      <div class="offset-by-three six columns">
        <br/>
        <h4>Get the spacemen ready!</h4>
        <input
          class="u-full-width"
          type="url"
          placeholder="Place your spacemen API link here..."
          id="urlSpacemen"
          v-model="urlSpacemen">
        <button :class="button.class" @click="toggle">{{button.text}}</button>
        <p v-if="failed"><em>Oups: {{errorMessage}}</em></p>
      </div>
    </div>

    <br/>
    <!-- Rockets view -->
    <div v-if="assigned" class="row">
      <label for="selectedShip">Your ships</label>
      <select v-model="selectedShip" id="selectedShip">
        <option v-for="spaceship in spaceships" :key="spaceship.name">
          {{spaceship.name}}
        </option>
      </select>
      <hr>

      <br/>

      <!-- Ship info -->
      <div v-for="spaceship in selectedFilter(spaceships)" :key="spaceship.name">
        <h3>{{spaceship.name}}</h3>
        <div v-for="member in spaceship.crew" :key="member.name">
          <hr>
          <h5>Name: <em>{{member.name}}</em></h5>
          <h5>Role: <em>{{member.role}}</em></h5>
          <h5>Age: <em>{{member.age}}</em></h5>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import Axios from "axios";
export default {
  name: "Home",
  data() {
    return {
      urlSpacemen: "",
      assigned: false,
      errorMessage: "",
      failed: false,
      spaceships: [],
      selectedShip: {},

      button: {
        class: "button-green",
        text: "Assign Rockets",
        // Constants
        ACTIVE_CLASS: "button-green",
        RESET_CLASS: "button-primary",
        ACTIVE_TEXT: "Assign Rockets",
        RESET_TEXT: "Reset"
      },

      // CONSTANTS
      SPACE_ROLES: [
        "Counselor",
        "Operations Officer",
        "Engineer",
        "Medical Officer",
        "Tactical Officer",
        "Alien Archaeologist",
        "Stellar Cartographer",
        "Security Officer"
      ]
    };
  },
  methods: {
    toggle() {
      if (!this.verifyToggle()) {
        return;
      }

      if (!this.assigned) {
        this.assignSeats();
      } else {
        this.reset();
      }
    },
    assignSeats() {
      this.button.text = this.button.RESET_TEXT;
      this.button.class = this.button.RESET_CLASS;
      this.assigned = true;

      // Handle
      const url = this.urlSpacemen;
      const app = this;
      // TODO: validate if url or not
      // if(!this.validateUrl(url)){
      //   this.setFailed(true,'Not a valid url');
      //   return;
      // }
      Axios.get(url) // grab data from this link
        .then(function(response) {
          console.dir(response);
          if (!response.data.spacemen) {
            throw new Error("The response is invalid");
          }
          if (response.data.spacemen.length == 0) {
            throw new Error("The list of spacemen is empty");
          }

          const spacemen = response.data.spacemen;

          // TODO: shuffle spacemen

          const boundAssignShips = app.assignSpaceShips.bind(app);
          boundAssignShips(spacemen);
          app.selectedShip = app.spaceships[0].name;
        })
        .catch(function(error) {
          // error occured case
          console.log(error);
          app.setFailed(true, "Could not get the spacemen...");
        });
    },
    reset() {
      this.button.text = this.button.ACTIVE_TEXT;
      this.button.class = this.button.ACTIVE_CLASS;
      this.assigned = false;

      // Reset
      this.spaceships = [];
    },
    assignSpaceShips(spacemen) {
      console.log(spacemen);

      // TODO: fix this for a nice grouping
      let subsetSize = Math.floor(spacemen.length / 3);
      subsetSize = subsetSize == 0 ? spacemen.length : subsetSize;
      const crew = spacemen.splice(0, subsetSize);

      console.log(crew);

      const newSpaceShip = this.produceSpaceShip(crew);
      this.spaceships.push(newSpaceShip);

      console.log(this.spaceships);
      if (spacemen.length == 0) {
        return;
      } else {
        this.assignSpaceShips(spacemen);
      }
    },
    produceSpaceShip(spacemen) {
      const crew = this.produceSpaceCrew(spacemen);
      console.log(crew);

      let shipName = crew
        .map(memb => {
          return memb.name.substr(0, 1).toUpperCase();
        })
        .join(".");
      // add random seed
      shipName += ' '+(Math.floor(Math.random() * (100)) + 100).toString();
      console.log(`Got ship name: ${shipName}`);
      return {
        name: shipName,
        crew: crew
      };
    },
    produceSpaceCrew(spacemen) {
      const captainIndex = Math.floor(Math.random() * spacemen.length);
      let captainSet = spacemen.splice(captainIndex, 1);
      captainSet[0].role = "Captain";
      console.log("Got captain");
      console.log(typeof captainSet[0].name);
      console.log(captainSet[0].name);

      spacemen.forEach(spacemen => {
        const randomRole = this.SPACE_ROLES[
          Math.floor(Math.random() * this.SPACE_ROLES.length)
        ];
        spacemen.role = randomRole;
      });
      const crew = captainSet.concat(spacemen);
      console.log("Produced crew");
      console.log(crew);
      return crew;
    },
    selectedFilter(spaceships) {
      const app = this;
      return spaceships.filter(function(ship) {
        return ship.name == app.selectedShip;
      });
    },
    setFailed(failed, msg) {
      this.failed = failed;
      this.errorMessage = msg;
    },
    verifyToggle() {
      if (this.urlSpacemen == "") {
        this.setFailed(true, "Url is not specified");
        return false;
      }
      this.setFailed(false, "");
      return true;
    }
  }
};
</script>
