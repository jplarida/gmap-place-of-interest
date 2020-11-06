<template>
  <div class="container mt-3">
        <div class="row">
            <div class="col-md-12">
                <h2>Search for places of interest near your preferred location</h2>
            </div>
        </div>
        <!--Row with two equal columns-->
        <div class="row mt-1">
            <div class="col-md-2">
                <label>Preferred location: </label>
            </div>
            <div class="col-md-10">
                <div class="demo-content bg-alt">
                  <vue-google-autocomplete class="form-control"
                        ref="address"
                        id="map"
                        classname="form-control"
                        placeholder="Please your location preference"
                        v-on:placechanged="getAddressData"
                    >
                  </vue-google-autocomplete>
                </div>
            </div>
        </div>
        
        <!--Row with two columns divided in 1:2 ratio-->
        <div class="row mt-1">
            <div class="col-md-2">
                <div class="demo-content">
                  <label>Keyword interest: </label>
                </div>
            </div>
            <div class="col-md-8">
                <div class="demo-content bg-alt">
                  <input type="text" class="form-control"  v-model="keywordsearch" :placeholder="placeholderValue">
                </div>
            </div>
            <div class="col-md-2">
                <div class="demo-content">
                  <button @click="getSearchKeyword" class="btn btn-info">Search</button>
                </div>
            </div>
        </div>
        
        <!--Row with two columns divided in 1:3 ratio-->
        <div class="row mt-4">
            <div class="col-md-12">
                <div id="mapdisplay" class="h-500" style="width:100%;  height: 600px;"></div>
            </div>
        </div>
  </div>
</template>

<script>
import VueGoogleAutocomplete from 'vue-google-autocomplete';
let service;
let map;
let infowindow ;
export default {
  name: "GoogleMap",
  components: { VueGoogleAutocomplete },
  data() {
    return {
      // default to Montreal to keep it simple
      // change this to whatever makes sense
      center: { lat: null, lng: null },
      markers: [],
      places: [],
      currentPlace: null,
      keywordsearch:null,
      zoom:15,
      address:null,
      placeholderValue: 'Keyword Search'
    };
  },
  mounted() {
    this.geolocate();
    this.$refs.address.focus();
  },

  methods: {
    // receives a place object via the autocomplete component
    setPlace(place) {
      this.currentPlace = place;
      this.center = {
          lat: this.currentPlace.geometry.location.lat(),
          lng: this.currentPlace.geometry.location.lng()
        };
    },
    getSearchKeyword: function(){
        this.searchKeyword();
    },
    addMarker() {
      if (this.currentPlace) {
        const marker = {
          lat: this.currentPlace.geometry.location.lat(),
          lng: this.currentPlace.geometry.location.lng()
        };
        this.markers.push({ position: marker });
        this.places.push(this.currentPlace);
        this.center = marker;
        this.currentPlace = null;
      }
    },
    getAddressData: function (addressData, placeResultData, id) {
      this.address = addressData;
      this.center = {
          lat: addressData.latitude,
          lng: addressData.longitude
        };
        this.loadMap();
    },
    geolocate: function() {
      if(!this.center.lat && !this.center.lng){
          navigator.geolocation.getCurrentPosition(position => {
            this.center = {
              lat: position.coords.latitude,
              lng: position.coords.longitude
            };
            this.loadMap();
          });
      }
      else
      {
        this.loadMap();
      }
    },
    loadMap(){
      map = new google.maps.Map(document.getElementById('mapdisplay'), {
       center: new google.maps.LatLng(this.center.lat, this.center.lng),
       zoom: this.zoom
     });
    },
    createMarker(place) {
        infowindow = new google.maps.InfoWindow();
        const marker = new google.maps.Marker({
          map,
          position: place.geometry.location,
        });
        google.maps.event.addListener(marker, "click", () => {
          infowindow.setContent(place.name);
          infowindow.open(map);
        });
    },
    searchKeyword(){
      if(this.keywordsearch){
        let request = {
          location: new google.maps.LatLng(this.center.lat, this.center.lng),
          rankBy: google.maps.places.RankBy.DISTANCE,
          keyword: this.keywordsearch
        };
        service = new google.maps.places.PlacesService(map);
        service.nearbySearch(request, (results, status) => {
          if (status === google.maps.places.PlacesServiceStatus.OK) {
              for (let i = 0; i < results.length; i++) {
              this.createMarker(results[i]);
              }
              map.setCenter(results[0].geometry.location);
          }
        });
      }
    }
  }
};
</script>