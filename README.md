# guaitaSKP

![Version](https://img.shields.io/badge/version-0.14-C6FF00?labelColor=0a0a0a)
![License](https://img.shields.io/badge/license-AGPL--3.0--only-C6FF00?labelColor=0a0a0a)
![Format](https://img.shields.io/badge/format-.skp%20only-C6FF00?labelColor=0a0a0a)

Visor 3D de fitxers **SketchUp (`.skp`)** al navegador, sense SketchUp. Part de la suite **Guaita Apps**.
A 3D viewer for **SketchUp (`.skp`)** files in the browser, no SketchUp needed. Part of the **Guaita Apps** suite.

[Català](#català) · [English](#english)

---

## Català

### Què és

guaitaSKP llegeix fitxers `.skp` i en reconstrueix la geometria, els colors dels materials i les **capes** (etiquetes) reals. Els mostra en un visor amb l'aspecte d'SketchUp: cel de fons, terra a cota 0 i la terna d'eixos vermell/verd/blau. Només admet `.skp`.

### Característiques

- **Diversos fitxers alhora**, per botó, tecla `O` o arrossegant-los al visor.
- **Selecció d'elements:** un clic selecciona tot el grup o component (no una cara solta); `Maj` + clic afegeix o treu. Es pot **aïllar**, **amagar** i **mostrar-ho tot**.
- **Capes reals del fitxer**, amb el seu color i nombre d'elements, que s'activen i desactiven amb una casella. Respecta les capes apagades als fitxers antics (anteriors a SketchUp 2021).
- **Caixa de secció** amb gir i límits en X, Y i alçada, i un tap de *poché* fosc a les cares que el tall deixa a la vista.
- **Aspecte SketchUp:** fons de cel per defecte (també blanc, gris fosc i negre; el fons triat es recorda), terra sòlid a cota 0 amb quadrícula, i eixos RGB a l'origen intern de cada fitxer.
- **Vistes:** planta, quatre alçats, axonometria, perspectiva i ortogràfica, enquadrar tot o la selecció.
- **Per model:** visibilitat, opacitat, color pla i aïllament.
- **Filferro** i **transparència** (des del menú).
- Menú clàssic en cascada i dreceres de teclat.

| Tecla | Acció |
|-------|-------|
| `O` | Obre fitxer(s) .skp |
| `F` | Enquadra tot o la selecció |
| `I` / `H` / `X` | Aïlla / amaga / mostra-ho tot |
| `Esc` | Deselecciona |

### Com s'usa

1. Descarrega `guaitaSKP 0.14.html` i obre'l en un navegador modern amb WebGL (o serveix-lo com a pàgina estàtica, per exemple amb GitHub Pages).
2. Arrossega un o més fitxers `.skp` al visor.
3. Gira amb el botó esquerre, desplaça amb el dret i fes zoom amb la roda.

El manual complet és a `HLP guaitaSKP-ajuda.html` (català) i `HLP guaitaSKP-en-ajuda.html` (anglès). Hi ha també una versió amb **interfície en anglès**, `guaitaSKP-en 0.14.html`, amb el mateix codi.

### Requisits i privadesa

- Els `.skp` es processen **localment al navegador**; no es pugen enlloc.
- El visor **necessita connexió en carregar-se**: Three.js (via jsDelivr) i els tipus de lletra IBM Plex (via Google Fonts) es descarreguen d'una CDN. No funciona completament offline, a diferència d'altres eines de la suite. El lector de `.skp` (OpenSKP) sí que va incrustat al fitxer.

### Limitacions conegudes

- **Sense textures:** cada cara es pinta amb el color pla del seu material.
- **Lectura per enginyeria inversa.** El format `.skp` no és públic; OpenSKP el reconstrueix sense garanties de cobertura total. S'ha provat amb fitxers de SketchUp 2017 a 2020; els de 2021 endavant no s'han provat en aquest visor. Algun fitxer antic poc habitual pot fallar amb un missatge d'error.
- **Diversos models:** cada un es centra pel seu compte (X/Z al voltant de l'origen, base a cota 0); no es conserva la posició relativa entre ells.
- **Sense fitxa de propietats:** seleccionar només ressalta l'element; no en mostra el nom ni la capa.
- **Controls encara inactius:** les dreceres `W` i `T` que mostra el menú no responen al teclat; els botons de la barra d'eines Filferro i Transparència, el botó Mesura i els dos botons de la fila «Estil» del quadre de vistes no fan res (el filferro i la transparència funcionen des del menú).
- **«Mostra-ho tot»** (`X`) reactiva també els elements de capes desmarcades, però les caselles de la llista no es tornen a marcar soles.
- **Models molt complexos** poden trigar a carregar: cada grup o component esdevé malles pròpies.

### Com està fet

Un únic fitxer HTML (~275 KB) amb el CSS, la lògica i el motor de lectura incrustats.

- **Lectura:** [OpenSKP](https://github.com/iamahsanmehmood/openskp) 1.3.0 (JS pur, MIT) i les seves dependències fflate, earcut i flatbuffers, empaquetats amb esbuild com a IIFE (~150 KB) i incrustats inline.
- **Renderitzat:** [Three.js](https://threejs.org) r160 amb `OrbitControls`, carregats per *import map*.
- **Base:** el nucli 3D (escena, seccions amb poché d'estergit, selecció per *handle* i panell de capes) s'ha reaprofitat de **guaitaFedera**, adaptat a un sol format.
- **Elements:** cada grup o component (identificat pel `path` d'OpenSKP) és un element amb un *handle* sintètic `índexModel·2³² + n`, amb colors de material com a colors de vèrtex. Amagar una capa és afegir-ne els *handles* al conjunt d'amagats, sense tornar a llegir el fitxer.
- **Coordenades:** OpenSKP dona geometria en metres i Y amunt; l'eix Z d'SketchUp passa a ser l'Y de Three.js i els colors de la terna en respecten l'orientació.

**Regenerar el bundle d'OpenSKP:**

```bash
npm install openskp@1.3.0 esbuild
echo "export { SkpFile, SkpParseError, toOBJ, toMTL, toGLB } from 'openskp';" > entry.js
npx esbuild entry.js --bundle --format=iife --global-name=OpenSKP \
  --minify --target=es2019 --platform=browser --outfile=openskp.bundle.js
```

> En injectar el bundle en un HTML amb `String.replace`, fes servir una **funció** com a substitut (`() => bundle`): el codi minificat conté seqüències amb `$` que `replace` interpreta com a patrons especials i corrompen el resultat.

### Fitxers

| Fitxer | Descripció |
|--------|------------|
| `guaitaSKP 0.14.html` | El visor (interfície en català) |
| `guaitaSKP-en 0.14.html` | El mateix visor amb interfície en anglès |
| `HLP guaitaSKP-ajuda.html` | Manual d'ús (català) |
| `HLP guaitaSKP-en-ajuda.html` | User manual (English) |
| `integracio-skp-federa.md` | Notes per incorporar `.skp` com a format de guaitaFedera |
| `LICENSE` | Text complet de la GNU AGPL v3 |
| `THIRD-PARTY-NOTICES.md` | Avisos i llicències dels components de tercers incrustats |

### Crèdits i llicència

Copyright © 2026 Jordi Subirós · SBS BIM Consulting. Llicència **GNU Affero General Public License v3.0 només** (`SPDX-License-Identifier: AGPL-3.0-only`); vegeu `LICENSE`.

**Components de tercers incrustats** al fitxer HTML (cada un conserva la seva llicència; els textos són a `THIRD-PARTY-NOTICES.md` i dins l'HTML):

| Component | Llicència | Titular |
|-----------|-----------|---------|
| [OpenSKP](https://github.com/iamahsanmehmood/openskp) 1.3.0 | MIT | Ahsan Mehmood |
| [fflate](https://github.com/101arrowz/fflate) 0.8.3 | MIT | Arjun Barrett |
| [earcut](https://github.com/mapbox/earcut) 3.2.3 | ISC | Mapbox |
| [flatbuffers](https://github.com/google/flatbuffers) 25.9.23 | Apache-2.0 | The FlatBuffers project |

**Carregats des de CDN en execució** (no es redistribueixen): Three.js (MIT) i IBM Plex (SIL OFL 1.1).

> Guaita Apps es distribueix «tal com és», sense cap garantia. Les eines estan pensades desde la recerca i com a suport tècnic i educatiu; la verificació dels resultats i la responsabilitat derivada del seu ús recauen únicament sobre l'usuari final.

---

## English

### What it is

guaitaSKP reads `.skp` files and rebuilds their geometry, material colours and real **layers** (tags). It shows them in a viewer styled after SketchUp: sky background, ground at elevation 0 and the red/green/blue axes triad. It only opens `.skp`.

### Features

- **Several files at once**, via button, the `O` key, or by dragging them onto the viewer.
- **Element selection:** one click selects the whole group or component (not a single face); `Shift` + click adds or removes. You can **isolate**, **hide** and **show all**.
- **The file's real layers**, with colour and element count, toggled with a checkbox. Layers switched off in older files (before SketchUp 2021) are respected.
- **Section box** with rotation and X, Y and height limits, and a dark *poché* cap on the faces the cut exposes.
- **SketchUp look:** sky background by default (also white, dark grey and black; your choice is remembered), a solid ground plane at elevation 0 with a grid, and RGB axes at each file's internal origin.
- **Views:** plan, four elevations, axonometric, perspective and orthographic, fit everything or the selection.
- **Per model:** visibility, opacity, flat colour and isolation.
- **Wireframe** and **transparency** (from the menu).
- Classic cascading menu and keyboard shortcuts.

| Key | Action |
|-----|--------|
| `O` | Open .skp file(s) |
| `F` | Fit everything or the selection |
| `I` / `H` / `X` | Isolate / hide / show all |
| `Esc` | Deselect |

### Usage

1. Download `guaitaSKP 0.14.html` and open it in a modern browser with WebGL (or serve it as a static page, e.g. with GitHub Pages).
2. Drag one or more `.skp` files onto the viewer.
3. Orbit with the left button, pan with the right, zoom with the wheel.

The full manual is in `HLP guaitaSKP-en-ajuda.html` (English) and `HLP guaitaSKP-ajuda.html` (Catalan). The interface of `guaitaSKP 0.14.html` is in Catalan; **`guaitaSKP-en 0.14.html` is the same viewer with an English interface**.

### Requirements and privacy

- `.skp` files are processed **locally in the browser**; nothing is uploaded.
- The viewer **needs a connection when it loads**: Three.js (via jsDelivr) and the IBM Plex fonts (via Google Fonts) are downloaded from a CDN. Unlike other tools in the suite, it does not work fully offline. The `.skp` reader (OpenSKP) is embedded in the file.

### Known limitations

- **No textures:** each face is painted with the flat colour of its material.
- **Reverse-engineered reading.** The `.skp` format is not public; OpenSKP rebuilds it with no guarantee of full coverage. Tested with SketchUp 2017–2020 files; 2021 and later have not been tested in this viewer. Some unusual older file may fail with an error message.
- **Several models:** each is centred on its own (X/Z around the origin, base at elevation 0); their relative position is not preserved.
- **No properties sheet:** selecting only highlights the element; it does not show its name or layer.
- **Inactive controls:** the `W` and `T` shortcuts shown in the menu do not respond to the keyboard; the toolbar's Wireframe and Transparency buttons, the Measure button and the two buttons in the "Estil" (Style) row of the views box do nothing (wireframe and transparency work from the menu).
- **"Mostra-ho tot" (Show all)** (`X`) also re-enables elements of unchecked layers, but the checkboxes in the list are not re-ticked automatically.
- **Very complex models** may be slow to load: each group or component becomes its own meshes.

### How it is built

A single HTML file (~275 KB) with the CSS, logic and reader engine embedded.

- **Reading:** [OpenSKP](https://github.com/iamahsanmehmood/openskp) 1.3.0 (pure JS, MIT) and its dependencies fflate, earcut and flatbuffers, bundled with esbuild as an IIFE (~150 KB) and inlined.
- **Rendering:** [Three.js](https://threejs.org) r160 with `OrbitControls`, loaded through an import map.
- **Foundation:** the 3D core (scene, sections with stencil poché caps, handle-based selection and layers panel) is reused from **guaitaFedera**, adapted to a single format.
- **Elements:** each group or component (identified by OpenSKP's `path`) is an element with a synthetic handle `modelIndex·2³² + n`, with material colours stored as vertex colours. Hiding a layer means adding its handles to the hidden set, without re-reading the file.
- **Coordinates:** OpenSKP outputs geometry in metres, Y-up; SketchUp's Z axis becomes Three.js's Y, and the triad colours follow that orientation.

**Regenerating the OpenSKP bundle:**

```bash
npm install openskp@1.3.0 esbuild
echo "export { SkpFile, SkpParseError, toOBJ, toMTL, toGLB } from 'openskp';" > entry.js
npx esbuild entry.js --bundle --format=iife --global-name=OpenSKP \
  --minify --target=es2019 --platform=browser --outfile=openskp.bundle.js
```

> When injecting the bundle into an HTML file with `String.replace`, use a **function** as the replacement (`() => bundle`): minified code contains `$` sequences that `replace` treats as special patterns, corrupting the output.

### Files

| File | Description |
|------|-------------|
| `guaitaSKP 0.14.html` | The viewer (Catalan interface) |
| `guaitaSKP-en 0.14.html` | The same viewer with an English interface |
| `HLP guaitaSKP-ajuda.html` | User manual (Catalan) |
| `HLP guaitaSKP-en-ajuda.html` | User manual (English) |
| `integracio-skp-federa.md` | Notes on adding `.skp` as a guaitaFedera format (Catalan) |
| `LICENSE` | Full text of the GNU AGPL v3 |
| `THIRD-PARTY-NOTICES.md` | Notices and licences of the embedded third-party components |

### Credits and licence

Copyright © 2026 Jordi Subirós · SBS BIM Consulting. Licensed under the **GNU Affero General Public License v3.0 only** (`SPDX-License-Identifier: AGPL-3.0-only`); see `LICENSE`.

**Third-party components embedded** in the HTML file (each keeps its own licence; the texts are in `THIRD-PARTY-NOTICES.md` and inside the HTML):

| Component | Licence | Copyright holder |
|-----------|---------|------------------|
| [OpenSKP](https://github.com/iamahsanmehmood/openskp) 1.3.0 | MIT | Ahsan Mehmood |
| [fflate](https://github.com/101arrowz/fflate) 0.8.3 | MIT | Arjun Barrett |
| [earcut](https://github.com/mapbox/earcut) 3.2.3 | ISC | Mapbox |
| [flatbuffers](https://github.com/google/flatbuffers) 25.9.23 | Apache-2.0 | The FlatBuffers project |

**Loaded from a CDN at runtime** (not redistributed): Three.js (MIT) and IBM Plex (SIL OFL 1.1).

> Guaita Apps es distribueix «tal com és», sense cap garantia. Les eines estan pensades desde la recerca i com a suport tècnic i educatiu; la verificació dels resultats i la responsabilitat derivada del seu ús recauen únicament sobre l'usuari final.
>
> *English translation, for convenience:* Guaita Apps is distributed "as is", without any warranty. The tools are designed from research and as technical and educational support; verification of the results and any liability arising from their use rest solely with the end user.
