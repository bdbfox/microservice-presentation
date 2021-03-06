
class: center, middle

.image[
![Fox DCG](images/fox-dcg-logo-updated.png)
]

# Microservices and Docker
.row[
.col[.image[
![Fox](images/docker-logo.png)
]]
.col[.image[
![FX](images/consul-logo.png)
]]
.col[.image[
![FX](images/lerna-logo.png)
]]
.col[.image[
![FX](images/terraform-logo.png)
]]
]

---
# Who is FOX DCG
.row[
.col[.image[
![Fox](images/fox-logo.png)
]]
.col[.image[
![FX](images/fx-logo.png)
]]
.col[.image[
![National Geographic](images/nat-geo-logo.png)
]]
.col[.image[
![FOX Sports](images/fox-sports-logo.png)
]]
]

- Responsible for consumer video applications across FOX, FOX Sports, FX, and National Geographic, ie. FOXNOW, FXNOW, etc...

- Build APIs and services for client applications across all platforms

### About me

- At FOX 1.5 years, at Disney/ABC before that almost 10 years.

- Background as a full stack developer.


---
### Goals

#### Content/Data management (CMS) needs to be based on the same services as the client apps.
 - Client (iOS/Android app/etc...) always sees the same thing as the editor in the CMS.
 So CMS is really just a React/Redux front end for the same services that the client app uses.

 - Data can be sourced can be from a variety of places, each with different levels of quality or rules.
 So we need a way to translate or update the content before it is used.

 - Ability to override a single field on any platform but still inherit other fields.

---
### Goals

#### Documentation needs to be simple and based on code.
 - Documentation is time consuming and will always get out of sync with code. So make it based on the code.

 - Exceptions happen so framework needs to be flexible to match.

 - Use Hypermedia driven web APIs, so that the apis are discoverable.

 - Products like apigee127 take similar but opposite approach.

---
##### Sample model:
```js
const SeriesModel = new Schema({
  approved: {
    type: Boolean,
  },
  contentFlags: {
    type: [String],
  },
  fullEpisodeCount: {
    type: Number,
  },
  previewVideo: {
    type: Collection,
    ref: 'Video',
    fields: ['@id', '@type', 'playerScreenUrl'],
  },
});
```

##### Sample draft (in) translation:
```js
class SeriesDraftTranslation extends Translation {
  contentFlags() {
    _.uniq(this._all('contentFlags').map(x => x.toLowerCase());
  }
}
```

##### Sample published (out) translation:
```js
class SeriesPublishedTranslation extends Translation {
  approved() {
    return this._get('cms.approved') && this._any('restriced') === false;
  }

  fullEpisodeCount() {
    return Video.count({ showId: this.id });
  }
}
```
---
##### Generates swagger
```json
{
  "swagger": "2.0",
  ...
  "paths": {
    "/series": {
      "get": {
        "produces": ["application/json"],
        "parameters": [{
          "name": "approved",
          "in": "query",
          "required": false,
          "type": "boolean"
        },
        ...
        "responses": "200": {
          "description": "Collection of Series objects",
          "headers": {
            ...
          },
          "schema": {
            "$ref": "#/definitions/SeriesCollection"
          }
        },
      }
    },
  },
  "definitions": {
    "Series": {
      "type": "object",
      "properties": {
        "@id": {
          "type": "string"
        },
        "approved": {
          "type": "boolean"
        },
        ...
      }
    }
  }
}
```
---
##### Generates response
```json
{
  "@platform": "default",
  "@id": "/series",
  "@context": "/context/core.jsonld",
  "@type": "Collection",
  "@populated": false,
  "itemsPerPage": 25,
  "totalItems": 201,
  "member": [{
    "@id": "/series/13-going-on-30_2004",
    "@type": "Series",
    "approved": true,
    "contentFlags": ["new", "expiring soon"],
    "fullEpisodeCount": 12,
    "previewVideo": {
      "@type": "Collection",
      ...
      "member": [{
        "@id": "/video/622258f933fc7f979a4358bb22a62d32",
        "@type": "Video"
      }]
    }
  },
  ...
  ]
}
```
---
### Goals

#### Documentation needs to be simple and based on code.
 - Documentation is time consuming and will always get out of sync with code. So make it based on the code.

 - Exceptions happen so framework needs to be flexible to match.

 - Use Hypermedia driven web APIs, so that the apis are discoverable.

 - Products like apigee127 take similar but opposite approach.

---
### Goals

#### Easy for developers to work with and deploy
 - 
