# Schéma des `conditionsReussite` (assertions de niveau)

Dernier maillon avant de coder le simulateur. Un niveau décrit sa réussite comme un **tableau
d'assertions** portant sur l'**état final attendu** du port — jamais sur la commande tapée.
Le serveur, après chaque commande, applique ces assertions sur l'`EtatPort` courant.

> **Pourquoi l'état final et pas la commande ?** Plusieurs solutions valides existent
> (`docker run -d --name site nginx` ≡ `docker create` + `docker start`). On valide le **résultat**,
> ce qui rend le jeu robuste et conforme au principe « niveaux = données ».

## Principes

1. `conditionsReussite` est un **tableau** ; le niveau est réussi quand **toutes** les assertions
   sont vraies (**ET logique**).
2. Chaque assertion est un objet **discriminé par `type`** + ses paramètres.
3. Chaque assertion porte un `message` optionnel (texte affiché/loggué quand elle **échoue**) —
   utile pour un feedback ciblé et pour les indices.
4. Les types « négatifs » (`...Absent`) évitent d'avoir à introduire un opérateur de négation.
5. Tout `type` inconnu = erreur de **données de niveau** (à détecter au chargement, pas au runtime).

## Structure commune

```jsonc
{
  "type": "<nom>",      // discriminateur
  "message": "…",       // optionnel : affiché si l'assertion échoue
  // … paramètres propres au type
}
```

## Types d'assertions

| `type` | Paramètres | Vrai quand… | Chap. |
| --- | --- | --- | --- |
| `conteneurExiste` | `nom`, `image?`, `statut?` (`RUNNING`\|`CREATED`\|`EXITED`) | un conteneur `nom` existe (et correspond à `image`/`statut` si fournis) | 1 |
| `conteneurAbsent` | `nom` | aucun conteneur nommé `nom` | 1 |
| `nombreConteneurs` | `statut?`, `op` (`=`\|`>=`\|`<=`), `valeur` | le nombre de conteneurs (filtré par `statut`) satisfait la comparaison | 1–2 |
| `imageExiste` | `nom`, `tag?` | l'image `nom[:tag]` est dans l'entrepôt | 1, 6 |
| `portExpose` | `hote`, `conteneur?`, `portConteneur?` | un mapping `hote:portConteneur` existe (lié à `conteneur` si fourni) | 2 |
| `envDefini` | `conteneur`, `cle`, `valeur?` | la variable `cle` est définie (= `valeur` si fournie) | 3 |
| `logsContiennent` | `conteneur`, `motif` | les logs simulés du conteneur contiennent `motif` | 3 |
| `volumeExiste` | `nom` | un volume `nom` existe | 4 |
| `volumeMonte` | `conteneur`, `volume`, `cible?` | `volume` est monté dans `conteneur` (sur `cible` si fournie) | 4 |
| `bindMont` | `conteneur`, `hote`, `cible` | un bind mount `hote → cible` existe dans `conteneur` | 4 |
| `donneesPersistees` | `volume`, `cle?` | les données écrites survivent après suppression du conteneur | 4 |
| `reseauExiste` | `nom` | un réseau `nom` existe | 5 |
| `conteneurSurReseau` | `conteneur`, `reseau` | `conteneur` est attaché à `reseau` | 5 |
| `resolutionNom` | `depuis`, `vers` | `depuis` peut résoudre `vers` par son nom (même réseau) | 5 |
| `imageConstruite` | `nom`, `tag?`, `instructions?` (liste requise, ex. `["EXPOSE","CMD"]`) | une image a été **buildée** depuis le Dockerfile (contient les instructions requises) | 6 |
| `serviceActif` | `nom`, `replicas?` | un service (compose/swarm) `nom` tourne (avec `replicas` si fourni) | 7, 9 |
| `fichierPresent` | `chemin`, `contientMotif?` | un fichier de l'éditeur existe (et contient `contientMotif`) | 6–8 |

> **Extensible** : ajouter un type = ajouter une entrée ici + un cas dans l'évaluateur + des tests.
> Aucun autre code de niveau n'est touché.

## JSON Schema (validation des données de niveau)

À utiliser pour valider les documents `niveaux` au chargement (ex. Ajv).

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://docker-harbor/schemas/conditionsReussite.json",
  "title": "conditionsReussite",
  "type": "array",
  "minItems": 1,
  "items": { "$ref": "#/$defs/assertion" },
  "$defs": {
    "statutConteneur": { "enum": ["RUNNING", "CREATED", "EXITED"] },
    "base": {
      "type": "object",
      "properties": {
        "type": { "type": "string" },
        "message": { "type": "string" }
      },
      "required": ["type"]
    },
    "assertion": {
      "allOf": [
        { "$ref": "#/$defs/base" },
        {
          "oneOf": [
            {
              "properties": {
                "type": { "const": "conteneurExiste" },
                "nom": { "type": "string" },
                "image": { "type": "string" },
                "statut": { "$ref": "#/$defs/statutConteneur" }
              },
              "required": ["type", "nom"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "conteneurAbsent" },
                "nom": { "type": "string" },
                "message": { "type": "string" }
              },
              "required": ["type", "nom"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "nombreConteneurs" },
                "statut": { "$ref": "#/$defs/statutConteneur" },
                "op": { "enum": ["=", ">=", "<="] },
                "valeur": { "type": "integer", "minimum": 0 },
                "message": { "type": "string" }
              },
              "required": ["type", "op", "valeur"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "imageExiste" },
                "nom": { "type": "string" },
                "tag": { "type": "string" },
                "message": { "type": "string" }
              },
              "required": ["type", "nom"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "portExpose" },
                "hote": { "type": "integer" },
                "conteneur": { "type": "string" },
                "portConteneur": { "type": "integer" },
                "message": { "type": "string" }
              },
              "required": ["type", "hote"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "envDefini" },
                "conteneur": { "type": "string" },
                "cle": { "type": "string" },
                "valeur": { "type": "string" },
                "message": { "type": "string" }
              },
              "required": ["type", "conteneur", "cle"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "logsContiennent" },
                "conteneur": { "type": "string" },
                "motif": { "type": "string" },
                "message": { "type": "string" }
              },
              "required": ["type", "conteneur", "motif"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "volumeExiste" },
                "nom": { "type": "string" },
                "message": { "type": "string" }
              },
              "required": ["type", "nom"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "volumeMonte" },
                "conteneur": { "type": "string" },
                "volume": { "type": "string" },
                "cible": { "type": "string" },
                "message": { "type": "string" }
              },
              "required": ["type", "conteneur", "volume"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "bindMont" },
                "conteneur": { "type": "string" },
                "hote": { "type": "string" },
                "cible": { "type": "string" },
                "message": { "type": "string" }
              },
              "required": ["type", "conteneur", "hote", "cible"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "donneesPersistees" },
                "volume": { "type": "string" },
                "cle": { "type": "string" },
                "message": { "type": "string" }
              },
              "required": ["type", "volume"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "reseauExiste" },
                "nom": { "type": "string" },
                "message": { "type": "string" }
              },
              "required": ["type", "nom"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "conteneurSurReseau" },
                "conteneur": { "type": "string" },
                "reseau": { "type": "string" },
                "message": { "type": "string" }
              },
              "required": ["type", "conteneur", "reseau"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "resolutionNom" },
                "depuis": { "type": "string" },
                "vers": { "type": "string" },
                "message": { "type": "string" }
              },
              "required": ["type", "depuis", "vers"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "imageConstruite" },
                "nom": { "type": "string" },
                "tag": { "type": "string" },
                "instructions": { "type": "array", "items": { "type": "string" } },
                "message": { "type": "string" }
              },
              "required": ["type", "nom"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "serviceActif" },
                "nom": { "type": "string" },
                "replicas": { "type": "integer", "minimum": 1 },
                "message": { "type": "string" }
              },
              "required": ["type", "nom"],
              "additionalProperties": false
            },
            {
              "properties": {
                "type": { "const": "fichierPresent" },
                "chemin": { "type": "string" },
                "contientMotif": { "type": "string" },
                "message": { "type": "string" }
              },
              "required": ["type", "chemin"],
              "additionalProperties": false
            }
          ]
        }
      ]
    }
  }
}
```

## Type TypeScript (côté back)

Union discriminée — l'évaluateur fait un `switch (a.type)` exhaustif.

```ts
export type StatutConteneur = "RUNNING" | "CREATED" | "EXITED";

export type Assertion =
  | { type: "conteneurExiste"; nom: string; image?: string; statut?: StatutConteneur; message?: string }
  | { type: "conteneurAbsent"; nom: string; message?: string }
  | { type: "nombreConteneurs"; statut?: StatutConteneur; op: "=" | ">=" | "<="; valeur: number; message?: string }
  | { type: "imageExiste"; nom: string; tag?: string; message?: string }
  | { type: "portExpose"; hote: number; conteneur?: string; portConteneur?: number; message?: string }
  | { type: "envDefini"; conteneur: string; cle: string; valeur?: string; message?: string }
  | { type: "logsContiennent"; conteneur: string; motif: string; message?: string }
  | { type: "volumeExiste"; nom: string; message?: string }
  | { type: "volumeMonte"; conteneur: string; volume: string; cible?: string; message?: string }
  | { type: "bindMont"; conteneur: string; hote: string; cible: string; message?: string }
  | { type: "donneesPersistees"; volume: string; cle?: string; message?: string }
  | { type: "reseauExiste"; nom: string; message?: string }
  | { type: "conteneurSurReseau"; conteneur: string; reseau: string; message?: string }
  | { type: "resolutionNom"; depuis: string; vers: string; message?: string }
  | { type: "imageConstruite"; nom: string; tag?: string; instructions?: string[]; message?: string }
  | { type: "serviceActif"; nom: string; replicas?: number; message?: string }
  | { type: "fichierPresent"; chemin: string; contientMotif?: string; message?: string };

export type ConditionsReussite = Assertion[];

// Esquisse de l'évaluateur : renvoie les assertions NON satisfaites.
export function evaluer(conditions: ConditionsReussite, etat: EtatPort): Assertion[] {
  return conditions.filter((a) => !assertionVraie(a, etat));
}
// Niveau réussi ⇔ evaluer(...).length === 0
```

## Exemples complets

### Ch. 1 — « Lance un conteneur nginx nommé site »

```json
[
  { "type": "conteneurExiste", "nom": "site", "image": "nginx", "statut": "RUNNING",
    "message": "Le bateau 'site' doit être construit à partir du plan nginx et naviguer." }
]
```

### Ch. 2 — « Expose le site sur le quai 8080 »

```json
[
  { "type": "conteneurExiste", "nom": "site", "statut": "RUNNING" },
  { "type": "portExpose", "hote": 8080, "conteneur": "site",
    "message": "Il faut une passerelle entre le quai 8080 et le bateau." }
]
```

### Ch. 5 — « Fais communiquer api et db sur un réseau privé »

```json
[
  { "type": "reseauExiste", "nom": "canal" },
  { "type": "conteneurSurReseau", "conteneur": "api", "reseau": "canal" },
  { "type": "conteneurSurReseau", "conteneur": "db", "reseau": "canal" },
  { "type": "resolutionNom", "depuis": "api", "vers": "db",
    "message": "api doit joindre db par son nom, sans passer par la ville." }
]
```

## Rappels d'implémentation

- **Validation au chargement** : passer chaque document `niveaux` dans le JSON Schema (Ajv) au
  démarrage/seed → échoue vite si un niveau est mal décrit.
- **Évaluation au runtime** : 100 % **côté serveur** (anti-triche), après chaque commande.
- **Indices** : le `message` des assertions non satisfaites alimente un feedback progressif,
  distinct des 3 paliers d'indices du niveau.
- **Tests** : pour chaque `type`, au moins un cas vrai et un cas faux (cf.
  [`commandes-par-chapitre.md`](./commandes-par-chapitre.md) → « Pour l'implémentation »).
