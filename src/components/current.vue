<template>
    <div>
      <button class="get-location-btn" @click="getLocation">Get Your Current Location</button>
      <div v-if="location" class="location-container">
        <p>{{ location }}</p>
        
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        location: null
      };
    },
    methods: {
      getLocation() {
        if (navigator.geolocation) {
          navigator.geolocation.getCurrentPosition(
            position => {
              const latitude = position.coords.latitude;
              const longitude = position.coords.longitude;
  
              // Reverse geocoding using Google Maps Geocoding API
              const geocodingAPI = `https://maps.googleapis.com/maps/api/geocode/json?latlng=${latitude},${longitude}&key=AIzaSyDq6q97H6uSGlKAmZOVfhh7c7B8lL05Jo4`;
  
              fetch(geocodingAPI)
                .then(response => response.json())
                .then(data => {
                  if (data.results.length > 0) {
                    const formattedAddress = data.results[0].formatted_address;
                    this.location = `Location: ${formattedAddress}, Latitude: ${latitude}, Longitude: ${longitude}`;
                  } else {
                    this.location = 'No location information available.';
                  }
                })
                .catch(error => {
                  console.log(error);
                  this.location = 'Failed to get location.';
                });
            },
            error => {
              console.log(error);
              this.location = 'Failed to get location.';
            }
          );
        } else {
          this.location = 'Geolocation is not supported by this browser.';
        }
      }
    }
  };
  </script>
  
  <style>
/* Button Styles */
.get-location-btn {
  background-color: #4CAF50; /* Green */
  border: none;
  color: white;
  padding: 10px 20px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 10px;
  cursor: pointer;
  border-radius: 4px;
}

.get-location-btn:hover {
  background-color: #45a049;
}

/* Location Container Styles */
.location-container {
  background-color: #f2f2f2;
  padding: 20px;
  border-radius: 4px;
  margin-top: 20px;
}


</style>