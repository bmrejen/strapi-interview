# Data and GraphQL specifications

## Models

### Planet

| Field       | Type    |
| ----------- | ------- |
| id          | integer |
| name        | string  |
| description | string  |
| code        | string  |
| picture_url | string  |

### Character

| Field       | Type     |
| ----------- | -------- |
| id          | integer  |
| name        | string   |
| description | string   |
| born_at     | datetime |
| picture_url | string   |

*Relations:*

- Each character lives on a planet ⇒ A Planet has many Characters
- A character can have friends ⇒ A Character has many Characters

    *NB*: A friendship is always in both ways. If A is friend with B then B is also friend with A.

## GraphQL service

### Types

- **`Planet`**
    - `id`
    - `name`: (min: 1 char, max: 20 char)
    - `description`: (min: 15 char, max: 300 char)
    - `code`: with format `AA-AAA-11` (ex: `YX-JGP-91`)
    - `pictureUrl`: url to the picture
    - `population`: number of characters on that planet
    - `characters`: returns the list of characters that live on this planet

        **arguments**:

        - `limit`: limit the number of characters returned (default: 5, max: 10)

- **`Character`**
    - `id`
    - `name`: (min: 1 char, max: 20 char)
    - `description`: (min: 15 char, max: 300 char)
    - `bornAt`: ISO datetime (e.g: 1970-01-01T00:00:00Z)
    - `pictureUrl`: url to the picture
    - `planet`: The planet the character lives on
    - `friends`: character's friends

        **arguments**:

        - `limit`: limit the number of characters returned (default: 5, max: 10)

    *NB*: the picture urls can be taken anywhere on the web, no image storing system is asked

### Queries

- `planets`: returns the list of all planets

    **arguments**:

    - `page`: page index (starting at 1) (default: 1, min: 1)
    - `pageSize`: number of items returned per page (default: 10, min: 1, max: 100)

    **response**: an array of **`Planet`**

    **example query**

    ```graphql
    query planets {
      planets(pageSize: 12, page: 3) {
        pagination {
          total
          page
          pageSize
        }
        nodes {
          id
          name
          description
          code
          pictureUrl
          population
          characters(limit: 3) {
            id
          }
        }
      }
    }
    ```

- `characters`: returns all the characters

    **arguments**:

    - `page`: page index (starting at 1) (default: 1, min: 1)
    - `pageSize`: number of items returned per page (default: 10, min: 1, max: 100)
    - `planet`: id of a **`Planet`**
    - `birthDate`: ISO date (e.g 1970-01-01) (1970-01-01 would match characters born between 1970-01-01T00:00:00Z and 1970-01-01T23:59:99Z)

    **response**: an array of **`Character`**

    **exemple query**

    ```graphql
    query characters {
      characters (page: 2, planet: 4) {
        pagination {
          total
          page
          pageSize
        }
        nodes {
          id
          name
          description
          bornAt
          pictureUrl
          planet {
            name
          }
          friends(limit: 3) {
            name
          }
        }
      }
    }
    ```

- `character`: returns a character

    **arguments**:

    - `id`: id of a **`Character`**

    **response**: a **`Character`**

    **exemple query**

    ```graphql
    query character {
      characters (id: 1) {
        id
        name
        description
        bornAt
        pictureUrl
        planet {
          name
        }
        friends(limit: 3) {
          name
        }
      }
    }
    ```

**Mutations**

- `createPlanet`: create a **`Planet`**

    **arguments**:

    - `planetInfo`:
        - `name`
        - `description`
        - `code`
        - `pictureUrl`

    **response**: a **`Planet`**

    **exemple query**

    ```graphql
    mutation createPlanet {
      createPlanet(planetInfo: {
        name: "Tatooine",
        description: "Tatooine is a sparsely inhabited circumbinary desert planet located in the galaxy's Outer Rim Territories",
        code: "XT-FOE-43",
        pictureUrl: "https://upload.wikimedia.org/wikipedia/en/6/6d/Tatooine_%28fictional_desert_planet%29.jpg"
      }) {
        id
        name
      }
    }
    ```

- `createCharacter`: create a **`Character`**

    **arguments**:

    - `characterInfo`:
        - `name`
        - `description`
        - `bornAt`
        - `planet`: `Planet` code
        - `pictureUrl`
        - `friends`: A list of **`Character`** ids

    **response**: a **`Character`**

    **exemple query**

    ```graphql
    mutation createCharacter {
      createCharacter(characterInfo: {
        name: "Chewbacca",
        description: "Chewbacca, known affectionately to his friends as Chewie, is a Wookiee male warrior, smuggler, mechanic, pilot, and resistance fighter.",
        bornAt: "1994-08-29T21:10:00Z",
        planet: "YX-JGP-91",
        pictureUrl: "https://upload.wikimedia.org/wikipedia/en/6/6d/Chewbacca-2-.jpg",
        friends: [2, 4, 5]
      }) {
        id
        name
      }
    }
    ```
