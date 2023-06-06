<template>
    <div>
      <input
        type="text"
        v-model="searchQuery"
        ref="searchInput"
        placeholder="Enter location"
        @keyup.enter="performSearch"
        @input="updateAutocomplete"
        autofocus
      />
  
      <button class="search-btn" @click="performSearch">Search</button>
      <div class="map-container">
        <div id="map"></div>
      </div>

    <div class="time">
      <div id="time-zone"></div>
        <div id="local-time"></div>
</div>

      <div>
        <h2>Search History</h2>
        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th>Select</th>
                <th>Query</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(query, index) in paginatedSearchHistory" :key="index">
                <td>
                  <input type="checkbox" v-model="selectedItems" :value="query" />
                </td>
                <td>{{ query }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="table-actions">
          <button class="delete-selected-btn" @click="deleteSelected" :disabled="selectedItems.length === 0">Delete Selected</button>
        </div>
        <div class="pagination">
          <button class="pagination-btn" @click="previousPage" :disabled="currentPage === 1">Previous</button>
          <button class="pagination-btn" @click="nextPage" :disabled="currentPage === totalPages">Next</button>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import { toRaw } from 'vue';
  
  export default {
    computed: {
      paginatedSearchHistory() {
        const startIndex = (this.currentPage - 1) * this.itemsPerPage;
        const endIndex = startIndex + this.itemsPerPage;
        return this.searchHistory.slice(startIndex, endIndex);
      },
      totalPages() {
        return Math.ceil(this.searchHistory.length / this.itemsPerPage);
      },
    },
    data() {
      return {
        searchQuery: '',
        map: null,
        markers: {},
        autocomplete: null,
        placesService: null,
        autocompleteInput: null,
        searchHistory: [],
        currentPage: 1,
        itemsPerPage: 10,
        selectedItems: [],
      };
    },
  
    mounted() {
      const script = document.createElement('script');
      script.src =
        'https://maps.googleapis.com/maps/api/js?key=AIzaSyDq6q97H6uSGlKAmZOVfhh7c7B8lL05Jo4&libraries=places&callback=initMap';
      script.defer = true;
      script.async = true;
      window.initMap = this.initMap;
      document.head.appendChild(script);
    },
  
    methods: {
      initMap() {
        this.map = new google.maps.Map(document.getElementById('map'), {
          center: { lat: 37.7749, lng: -122.4194 },
          zoom: 12,
        });
  
        this.autocompleteInput = this.$refs.searchInput;
        this.autocomplete = new google.maps.places.Autocomplete(this.autocompleteInput);
        this.placesService = new google.maps.places.PlacesService(this.map);
  
        this.autocomplete.addListener('place_changed', this.onPlaceChanged);
      },
  
      updateAutocomplete() {
        this.autocomplete.setFields(['formatted_address', 'geometry']);
      },
  
      onPlaceChanged() {
        const place = this.autocomplete.getPlace();
        if (place.geometry) {
          this.searchQuery = place.formatted_address;
        }
      },
  
      performSearch() {
  if (
    this.searchQuery.trim() !== '' &&
    !this.$refs.searchInput.contains(document.activeElement)
  ) {
    const request = {
      query: this.searchQuery,
      fields: ['name', 'geometry'],
    };

    this.placesService.findPlaceFromQuery(request, (results, status) => {
      if (
        status === google.maps.places.PlacesServiceStatus.OK &&
        results &&
        results.length
      ) {
        const place = results[0];
        if (place.geometry) {
          this.searchQuery = place.formatted_address;
          const marker = new google.maps.Marker({
            map: this.map,
            position: place.geometry.location,
          });
          this.markers[place.name] = marker;
          this.map.panTo(place.geometry.location);
          this.map.setZoom(15);

          this.searchHistory.unshift(place.name);

          // Get time zone and local time
          const timestamp = Math.floor(Date.now() / 1000); // Current time in seconds
          const location = place.geometry.location;

          // Make a request to Time Zone API
          fetch(
            `https://maps.googleapis.com/maps/api/timezone/json?location=${location.lat()},${location.lng()}&timestamp=${timestamp}&key=AIzaSyDq6q97H6uSGlKAmZOVfhh7c7B8lL05Jo4`
          )
            .then((response) => response.json())
            .then((data) => {
              const timeZoneId = data.timeZoneId;
              const timeZoneOffset = data.rawOffset + data.dstOffset;
              const localTime = new Date(timestamp * 1000 + timeZoneOffset * 1000);

              const options = {
                timeZone: timeZoneId,
                hour12: true,
                hour: 'numeric',
                minute: 'numeric',
                second: 'numeric',
              };

              const localTimeString = localTime.toLocaleString('en-US', options);

              document.getElementById('time-zone').textContent = 'Time Zone: ' + timeZoneId;
              document.getElementById('local-time').textContent = 'Local Time: ' + localTimeString;
            })
            .catch((error) => {
              console.log('Unable to retrieve time zone information.', error);
            });
        }
      } else if (
        status === google.maps.places.PlacesServiceStatus.OK &&
        results &&
        results.length === 0
      ) {
        console.log('No results found for the entered location.');
      } else {
        console.log('Places search request failed with status:', status);
      }
    });
  }
},

  
      deleteSelected() {
        this.selectedItems.forEach((query) => {
          const marker = this.markers[query];
          if (marker) {
            const rawMarker = toRaw(marker);
            rawMarker.setMap(null); // Remove the marker from the map
            delete this.markers[query]; // Remove the marker from the markers object
          }
          const index = this.searchHistory.indexOf(query);
          if (index !== -1) {
            this.searchHistory.splice(index, 1); // Remove the query from the searchHistory array
          }
        });
        this.selectedItems = [];
      },
  /*
      toggleAllMarkers() {
        Object.values(this.markers).forEach((marker) => {
          marker.setVisible(!marker.getVisible());
        });
      },
  
      deleteAllMarkers() {
        Object.values(this.markers).forEach((marker) => {
          const rawMarker = toRaw(marker);
          rawMarker.setMap(null);
        });
        this.markers = {};
      },
  */
      previousPage() {
        if (this.currentPage > 1) {
          this.currentPage--;
        }
      },
  
      nextPage() {
        if (this.currentPage < this.totalPages) {
          this.currentPage++;
        }
      },
    },
  };
  </script>
  
  <style>
/* Input Styles */
input {
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-right: 10px;
}

/* Search Button Styles */
.search-btn {
  background-color: #4CAF50; /* Green */
  border: none;
  color: white;
  padding: 10px 20px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin-bottom: 10px;
  cursor: pointer;
  border-radius: 4px;
}

.search-btn:hover {
  background-color: #45a049;
}

/* Map Container Styles */
.map-container {
  height: 400px;
  margin-bottom: 20px;
}

#map {
  height: 100%;
}

/* Table Styles */
table {
  width: 100%;
  border-collapse: collapse;
  background-color: #fff;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

th,
td {
  padding: 12px 15px;
  text-align: left;
}

th {
  background-color: #f9f9f9;
}

tbody tr:nth-child(even) {
  background-color: #f5f5f5;
}

/* Pagination Styles */
.pagination {
  margin-top: 10px;
  display: flex;
  justify-content: center;
}

.pagination-btn {
  padding: 8px 12px;
  border-radius: 4px;
  background-color: #007bff;
  color: #fff;
  border: none;
  cursor: pointer;
  transition: background-color 0.3s ease;
  margin-right: 5px;
}

.pagination-btn:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.pagination-btn:not(:disabled):hover {
  background-color: #0056b3;
}

/* Table Actions Styles */
.table-actions {
  display: flex;
  justify-content: flex-end;
  margin-top: 10px;
}

.delete-selected-btn {
  background-color: #dc3545; /* Red */
  border: none;
  color: white;
  padding: 10px 20px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  cursor: pointer;
  border-radius: 4px;
}

.delete-selected-btn:hover {
  background-color: #c82333;
}

/* Table Container Styles */
.table-container {
  max-height: 200px;
  overflow-y: auto;
}
</style>