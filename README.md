<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Standort- und Passwortprüfung</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 20px;
    }
    .hidden {
      display: none;
    }
    .zone-status {
      margin-top: 20px;
      color: red;
    }
    .in-zone {
      color: green;
    }
  </style>
</head>
<body>
  <h1>Standort- und Passwortprüfung</h1>
  <p>Diese Website überprüft, ob du dich in einer der beiden Zonen befindest.</p>

  <button id="checkLocation">Standort überprüfen</button>
  <p id="locationStatus">Standortstatus: Unbekannt</p>

  <div id="zone1Actions" class="hidden">
    <button id="generatePassword">Passwort generieren</button>
    <p id="password">Passwort: Nicht generiert</p>
  </div>

  <div id="zone2Actions" class="hidden">
    <p>Gib das Passwort ein, das in Zone 1 generiert wurde:</p>
    <input type="text" id="inputPassword" placeholder="Passwort eingeben">
    <button id="submitPassword">Passwort prüfen</button>
    <p id="passwordFeedback"></p>
  </div>

  <p class="zone-status" id="zoneStatus">Bitte begib dich in eine gültige Zone.</p>

  <script>
    // Koordinaten der Zonen (in Dezimalgraden)
    const zone1 = { lat: 51.363167, lon: 12.407650, radius: 40 }; // Zone 1: Passwort wird generiert
    const zone2 = { lat: 51.370783, lon: 12.408583, radius: 40 }; // Zone 2: Passwort kann eingegeben werden

    // Globale Variablen
    let currentPassword = null; // Speichert das generierte Passwort
    let someoneInZone1 = false; // Status: Ist jemand in Zone 1?

    // Entfernung zwischen zwei Koordinaten berechnen
    const calculateDistance = (lat1, lon1, lat2, lon2) => {
      const toRad = (value) => (value * Math.PI) / 180;
      const R = 6371e3; // Erdradius in Metern
      const φ1 = toRad(lat1);
      const φ2 = toRad(lat2);
      const Δφ = toRad(lat2 - lat1);
      const Δλ = toRad(lon2 - lon1);
      const a = Math.sin(Δφ / 2) * Math.sin(Δφ / 2) +
                Math.cos(φ1) * Math.cos(φ2) *
                Math.sin(Δλ / 2) * Math.sin(Δλ / 2);
      const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
      return R * c; // Entfernung in Metern
    };

    const isWithinZone = (lat, lon, zone) => {
      const distance = calculateDistance(lat, lon, zone.lat, zone.lon);
      return distance <= zone.radius;
    };

    // Standort prüfen
    document.getElementById('checkLocation').addEventListener('click', () => {
      if (!navigator.geolocation) {
        alert('Geolocation wird von deinem Browser nicht unterstützt.');
        return;
      }

      navigator.geolocation.getCurrentPosition((position) => {
        const { latitude, longitude } = position.coords;
        document.getElementById('locationStatus').textContent = 
          `Dein Standort: Lat ${latitude.toFixed(5)}, Lon ${longitude.toFixed(5)}`;

        let inZone1 = isWithinZone(latitude, longitude, zone1);
        let inZone2 = isWithinZone(latitude, longitude, zone2);

        if (inZone1) {
          document.getElementById('zoneStatus').textContent = 'Du bist in Zone 1.';
          document.getElementById('zoneStatus').className = 'zone-status in-zone';
          document.getElementById('zone1Actions').classList.remove('hidden');
          someoneInZone1 = true; // Signalisiert, dass jemand in Zone 1 ist
        } else {
          document.getElementById('zone1Actions').classList.add('hidden');
        }

        if (inZone2) {
          document.getElementById('zoneStatus').textContent = 'Du bist in Zone 2.';
          document.getElementById('zoneStatus').className = 'zone-status in-zone';
          document.getElementById('zone2Actions').classList.remove('hidden');

          // Prüfen, ob jemand in Zone 1 ist
          if (!someoneInZone1) {
            document.getElementById('passwordFeedback').textContent = 
              'Es ist niemand in Zone 1. Bitte warte.';
          } else {
            document.getElementById('passwordFeedback').textContent = '';
          }
        } else {
          document.getElementById('zone2Actions').classList.add('hidden');
        }

        if (!inZone1 && !inZone2) {
          document.getElementById('zoneStatus').textContent = 
            'Bitte begib dich in eine gültige Zone.';
          document.getElementById('zoneStatus').className = 'zone-status';
        }
      });
    });

    // Passwort generieren
    document.getElementById('generatePassword').addEventListener('click', () => {
      currentPassword = Math.random().toString(36).slice(-8);
      document.getElementById('password').textContent = `Passwort: ${currentPassword}`;
    });

    // Passwort prüfen
    document.getElementById('submitPassword').addEventListener('click', () => {
      const enteredPassword = document.getElementById('inputPassword').value;
      if (enteredPassword === currentPassword) {
        document.getElementById('passwordFeedback').textContent = 
          'Passwort korrekt! Zugriff erlaubt.';
        document.getElementById('passwordFeedback').style.color = 'green';
      } else {
        document.getElementById('passwordFeedback').textContent = 
          'Passwort falsch. Bitte erneut versuchen.';
        document.getElementById('passwordFeedback').style.color = 'red';
      }
    });
  </script>
</body>
</html>
