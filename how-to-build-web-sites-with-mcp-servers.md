# How to Build Web Sites with MCP Servers

Tämä dokumentti kuvaa, miten **uutisseuranta**-projektin GitHub Pages -sivusto ja Firebase-integraatio rakennettiin käyttäen AI-assistenttia ja MCP (Model Context Protocol) -servereä.

---

## 1. GitHub Pages -sivuston luominen MCP:llä

### Lähtökohta

Projektilla oli olemassa oleva repositorio [`jaakkokorhonen/uutisseuranta`](https://github.com/jaakkokorhonen/uutisseuranta). Tavoitteena oli luoda projektille GitHub Pages -sivu.

### Miten se tehdään

AI-assistentti käytti **GitHub MCP -serveriä** luodakseen `index.html`-tiedoston suoraan repositorioon ilman, että käyttäjän tarvitsi itse koskea koodiin tai Git-komentoihin.

**Prosessi:**

1. AI luki repositorion rakenteen `get_file_contents`-työkalulla
2. Rakensi täysin toimivan, suomenkielisen `index.html`-sivun, joka sisältää:
   - Oma SVG-logo
   - Vaalea/tumma teema (automaattinen + manuaalinen vaihto)
   - Responsiivinen navigaatio
   - Heroj-osio tilastopalkeilla
   - Uutisvirta-esimerkki (2+2 grid)
   - Ominaisuusosio live-aktiivisuusvisualisoinnilla
   - Aihepiiriverkko (12 aihetta)
   - Avoin lähdekoodi -CTA
   - Alatunniste
3. Pushasi tiedoston `create_or_update_file`-työkalulla suoraan `main`-haaraan

**Sivun käyttöönotto GitHubissa (manuaalinen askel):**

1. Mene repositorion **Settings → Pages**
2. Valitse Source: `Deploy from a branch`
3. Valitse haara: `main`, kansio: `/ (root)`
4. Klikkaa **Save**

Sivusto on sen jälkeen osoitteessa `https://jaakkokorhonen.github.io/uutisseuranta/`.

---

## 2. Firebase Google Auth -integraatio

### Lähtökohta

Käyttäjä tarjosi Firebase-projektin konfiguraation:

```js
const firebaseConfig = {
  apiKey: "AIzaSyApRi0p3KXOe6W6F-t8QInqJoZQdjOfCjI",
  authDomain: "uutisseuranta-net.firebaseapp.com",
  projectId: "uutisseuranta-net",
  storageBucket: "uutisseuranta-net.firebasestorage.app",
  messagingSenderId: "131558328064",
  appId: "1:131558328064:web:2b1eabe45fdb807c9d55e5",
  measurementId: "G-9J9T62LY57"
};
```

### Miten se tehdään

AI päivitti `index.html`-tiedoston Firebase Auth -integraatiolla käyttäen GitHub MCP -serveriä:

1. Luki nykyisen `index.html`:n SHA-tunnisteen `get_file_contents`-työkalulla
2. Rakensi päivitetyn version, joka sisältää:
   - **Kirjautumismodaali** — Google-painike, sulkeutuu Escape-näppäimellä tai taustaa klikkaamalla
   - **Auth-tila navigaatiossa** — kirjautumisnapin tilalle profiilikuva, nimi ja `Ulos`-nappi
   - **`onAuthStateChanged`** — session automaattinen palautus sivulatauksen yhteydessä
   - Firebase SDK ladataan ES-moduuleina suoraan Googlen CDN:ltä (ei build-steppiä)
3. Pushasi päivityksen SHA:n kera `create_or_update_file`-työkalulla

**Firebase-konsolissa tarvittava manuaalinen askel:**

Google-kirjautuminen vaatii, että sivuston domain on sallittu:

1. Mene [Firebase Console → Authentication → Settings → Authorized domains](https://console.firebase.google.com/project/uutisseuranta-net/authentication/settings)
2. Klikkaa **Add domain**
3. Lisää: `jaakkokorhonen.github.io`

Ilman tätä popup-kirjautuminen heittää `auth/unauthorized-domain`-virheen.

### Firebase SDK ilman build-steppiä

GitHub Pages on staattinen — ei Node.js:n, Webpackin tai muun build-työkalun tukea. Firebase toimii silti käyttämällä ES-moduuleja suoraan selaimen `type="module"` -skriptissä:

```html
<script type="module">
  import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js';
  import { getAuth, signInWithPopup, GoogleAuthProvider, onAuthStateChanged, signOut }
    from 'https://www.gstatic.com/firebasejs/10.12.0/firebase-auth.js';

  // ... loput koodista
</script>
```

---

## 3. Firebase MCP -serverin käyttöönotto

Firebasella on virallinen MCP-serveri osana `firebase-tools`-pakettia. Sen avulla AI-assistentti voi hallita Firebase-projektia suoraan (Firestore, Auth-käyttäjät, Rules, jne.).

### Esivalmistelu: kirjaudu Firebase CLI:llä

```bash
npx -y firebase-tools@latest login
```

### Claude Desktop

Muokkaa tiedostoa `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "firebase": {
      "command": "npx",
      "args": ["-y", "firebase-tools@latest", "experimental:mcp"]
    }
  }
}
```

Käynnistä Claude Desktop uudelleen.

### Cursor

Luo projektin juureen `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "firebase": {
      "command": "npx",
      "args": ["-y", "firebase-tools@latest", "experimental:mcp"]
    }
  }
}
```

### VS Code (Copilot / Cline)

Lisää `~/Library/Application Support/Code/User/settings.json`-tiedostoon sama `mcpServers`-konfiguraatio.

### Huomioita

- Serveri lukee automaattisesti hakemiston `firebase.json`-tiedoston ja yhdistää oikeaan projektiin
- Yli 30 työkalua: Firestore-kyselyt, Auth-käyttäjät, Rules-muokkaus, Functions-deploy jne.
- GitHub MCP + Firebase MCP yhdessä = täysi stack hallittavissa AI-assistentin kautta

---

## Yhteenveto: GitHub MCP -työkalut tässä projektissa

| Työkalu | Käyttötarkoitus |
|---|---|
| `get_file_contents` | Lue tiedoston sisältö ja SHA |
| `create_or_update_file` | Luo tai päivitä tiedosto (SHA vaaditaan päivitykseen) |
| `list_branches` | Tarkista haarat |
| `create_branch` | Luo uusi haara |
| `push_files` | Pushaa useita tiedostoja kerralla |
| `create_pull_request` | Avaa pull request |
| `merge_pull_request` | Mergaa PR |

> **Vinkki:** `create_or_update_file` vaatii olemassa olevan tiedoston päivitykseen aina edellisen version SHA:n. Hae se ensin `get_file_contents`-kutsussa.
