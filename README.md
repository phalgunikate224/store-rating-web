<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Store Rating Web Application</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');
  * {
    box-sizing: border-box;
  }
  body {
    font-family: 'Poppins', sans-serif;
    background: #f0f4f8;
    margin: 0;
    padding: 20px;
    color: #333;
  }
  h1 {
    text-align: center;
    font-weight: 600;
    margin-bottom: 1.5rem;
    color: #2a4365;
  }
  .container {
    max-width: 720px;
    margin: 0 auto;
    background: #fff;
    border-radius: 14px;
    box-shadow: 0 14px 35px rgba(0,0,0,0.1);
    padding: 30px 40px 40px;
  }
  .store-list {
    list-style: none;
    padding: 0;
    margin: 0;
  }
  .store {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 14px 12px;
    border-radius: 12px;
    transition: background-color 0.3s ease;
    margin-bottom: 16px;
    background: #e8f0fe;
  }
  .store:hover {
    background: #d1defc;
  }
  .store-name {
    font-size: 1.15rem;
    font-weight: 600;
    max-width: 50%;
  }
  .stars {
    display: flex;
  }
  .stars button {
    background: none;
    border: none;
    cursor: pointer;
    padding: 0 4px;
    font-size: 24px;
    color: #ccc;
    transition: color 0.2s ease;
    outline-offset: 4px;
    user-select: none;
  }
  .stars button.filled {
    color: #f6c90e;
    filter: drop-shadow(0 0 1px #a67a00);
  }
  .rating-info {
    margin-top: 6px;
    font-size: 0.9rem;
    color: #555;
  }
  .clear-ratings {
    margin-top: 20px;
    display: block;
    background: #e53e3e;
    color: white;
    border: none;
    border-radius: 10px;
    padding: 12px 20px;
    font-weight: 600;
    cursor: pointer;
    box-shadow: 0 4px 12px rgba(229,62,62,0.35);
    transition: background-color 0.25s ease;
    width: 100%;
    max-width: 300px;
    margin-left: auto;
    margin-right: auto;
  }
  .clear-ratings:hover {
    background: #c53030;
  }
  footer {
    margin-top: 40px;
    text-align: center;
    font-size: 0.9rem;
    color: #777;
  }
  @media (max-width: 480px) {
    .store-name {
      max-width: 40%;
      font-size: 1rem;
    }
    .stars button {
      font-size: 20px;
      padding: 0 2px;
    }
  }
</style>
</head>
<body>
  <h1>Store Rating</h1>
  <div class="container">
    <ul class="store-list" id="storeList">
      <!-- Stores will be populated here -->
    </ul>
    <button class="clear-ratings" id="clearRatingsBtn" title="Clear all stored ratings">Clear All Ratings</button>
  </div>
  <footer>Store Rating Web App &copy; 2024</footer>
  <script>
    (function(){
      const stores = [
        { id: 'store-1', name: 'Tech Haven' },
        { id: 'store-2', name: 'Book Nook' },
        { id: 'store-3', name: 'Fashion Factory' },
        { id: 'store-4', name: 'Gourmet Grocery' },
        { id: 'store-5', name: 'Sports Corner' }
      ];

      const maxStars = 5;
      const storeListEl = document.getElementById('storeList');
      const clearBtn = document.getElementById('clearRatingsBtn');

      // Retrieve ratings from localStorage or initialize empty object
      function getRatings() {
        const data = localStorage.getItem('storeRatings');
        return data ? JSON.parse(data) : {};
      }

      // Save ratings object to localStorage
      function setRatings(ratings) {
        localStorage.setItem('storeRatings', JSON.stringify(ratings));
      }

      // Calculate average rating for a given store ratings array
      function calcAverage(ratingsArr) {
        if (!ratingsArr || ratingsArr.length === 0) return 0;
        const sum = ratingsArr.reduce((a, b) => a + b, 0);
        return (sum / ratingsArr.length);
      }

      // Create star button element
      function createStarBtn(storeId, starIndex, filled) {
        const btn = document.createElement('button');
        btn.setAttribute('aria-label', `${starIndex + 1} star`);
        btn.setAttribute('role', 'radio');
        btn.setAttribute('tabindex', '0');
        btn.innerHTML = '&#9733;'; // Unicode star
        if (filled) {
          btn.classList.add('filled');
          btn.setAttribute('aria-checked', 'true');
        } else {
          btn.setAttribute('aria-checked', 'false');
        }
        btn.addEventListener('mouseover', () => {
          highlightStars(storeId, starIndex);
        });
        btn.addEventListener('mouseout', () => {
          resetStarsUI(storeId);
        });
        btn.addEventListener('click', () => {
          rateStore(storeId, starIndex + 1);
        });
        btn.addEventListener('keydown', (e) => {
          if (e.key === 'Enter' || e.key === ' ') {
            e.preventDefault();
            rateStore(storeId, starIndex + 1);
          }
        });
        return btn;
      }

      // Highlight stars on hover up to starIndex
      function highlightStars(storeId, starIndex) {
        const starBtns = document.querySelectorAll(`#${storeId} .stars button`);
        starBtns.forEach((btn, idx) => {
          if (idx <= starIndex) {
            btn.classList.add('filled');
          } else {
            btn.classList.remove('filled');
          }
        });
      }

      // Reset stars display to reflect actual average rating
      function resetStarsUI(storeId) {
        const ratings = getRatings();
        const storeRatings = ratings[storeId] || [];
        const avg = calcAverage(storeRatings);
        const starBtns = document.querySelectorAll(`#${storeId} .stars button`);
        starBtns.forEach((btn, idx) => {
          if (idx < Math.round(avg)) {
            btn.classList.add('filled');
            btn.setAttribute('aria-checked', 'true');
          } else {
            btn.classList.remove('filled');
            btn.setAttribute('aria-checked', 'false');
          }
        });
      }

      // Rate the store by adding the new rating and updating display
      function rateStore(storeId, rating) {
        let ratings = getRatings();
        if (!ratings[storeId]) {
          ratings[storeId] = [];
        }
        ratings[storeId].push(rating);
        setRatings(ratings);
        updateStoreUI(storeId);
      }

      // Update UI elements of a store after rating changes
      function updateStoreUI(storeId) {
        const ratings = getRatings();
        const storeRatings = ratings[storeId] || [];
        const avg = calcAverage(storeRatings).toFixed(2);
        const ratingsCount = storeRatings.length;

        // Update stars displayed
        resetStarsUI(storeId);

        // Update rating info text
        const ratingInfoEl = document.querySelector(`#${storeId} .rating-info`);
        ratingInfoEl.textContent = `Average rating: ${avg} (${ratingsCount} ${ratingsCount === 1 ? 'vote' : 'votes'})`;
      }

      // Render store list
      function renderStores() {
        storeListEl.innerHTML = '';
        stores.forEach(store => {
          const li = document.createElement('li');
          li.className = 'store';
          li.id = store.id;

          const nameSpan = document.createElement('span');
          nameSpan.className = 'store-name';
          nameSpan.textContent = store.name;

          const starsDiv = document.createElement('div');
          starsDiv.className = 'stars';
          starsDiv.setAttribute('role', 'radiogroup');
          starsDiv.setAttribute('aria-label', `Rate store ${store.name}`);

          // Create star buttons
          for (let i = 0; i < maxStars; i++) {
            starsDiv.appendChild(createStarBtn(store.id, i, false));
          }

          const ratingInfo = document.createElement('div');
          ratingInfo.className = 'rating-info';
          ratingInfo.textContent = 'No ratings yet';

          li.appendChild(nameSpan);
          li.appendChild(starsDiv);
          li.appendChild(ratingInfo);

          storeListEl.appendChild(li);
        });
      }

      // Initialize the app UI and ratings
      function init() {
        renderStores();
        stores.forEach(store => updateStoreUI(store.id));
      }

      // Clear all stored ratings (localStorage)
      function clearAllRatings() {
        if (confirm('Are you sure you want to clear all ratings?')) {
          localStorage.removeItem('storeRatings');
          init();
        }
      }
      clearBtn.addEventListener('click', clearAllRatings);

      init();
    })();
  </script>
</body>
</html>

