# Pokedex App

Este es un proyecto de una aplicación de Pokedex construida con React. La aplicación permite a los usuarios buscar, visualizar y explorar información detallada sobre diferentes Pokémon, incluidas sus estadísticas, evoluciones y otros datos relevantes.

Utiliza la api de [https://pokeapi.co/docs/v2|PokeApi]

## Características

- **Búsqueda de Pokémon**: Busca Pokémon por su nombre.
- **Paginación**: Navega a través de diferentes páginas de Pokémon.
- **Detalles del Pokémon**: Muestra detalles como altura, peso, hábitat, estadísticas y evoluciones del Pokémon.
- **Interfaz de Usuario Interactiva**: Utiliza React y SASS para una UI dinámica y atractiva.

## Requisitos

- Node.js
- npm o yarn

## Instalación

1. Clona el repositorio:
    ```sh
    git clone https://github.com/tu-usuario/pokedex-app.git
    cd pokedex-app
    ```

2. Instala las dependencias:
    ```sh
    npm install
    # o
    yarn install
    ```

3. Inicia la aplicación:
    ```sh
    npm start
    # o
    yarn start
    ```

## Estructura del Proyecto

- **`src`**: Contiene el código fuente de la aplicación.
  - **`components`**: Componentes reutilizables de React.
  - **`pages`**: Páginas de la aplicación.
  - **`styles`**: Archivos SASS para el estilizado de la aplicación.
  - **`api`**: Configuración de las URLs de la API.

## Descripción de Archivos Clave

### `main.jsx`

Este es el punto de entrada de la aplicación React. Renderiza el componente raíz `<App />` dentro del modo estricto de React.

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App.jsx';
import './styles/index.scss';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
);
```

### `App.jsx`

Este componente sirve como el contenedor principal para la aplicación y renderiza el componente `LayoutHome`.

```jsx
import { useState } from "react";
import LayoutHome from "./pages/home/layout/LayoutHome";

function App() {
  return (
    <div>
      <LayoutHome />
    </div>
  );
}

export default App;
```

### `Card.jsx`

Este componente es responsable de mostrar la información detallada de cada Pokémon. Utiliza varios hooks de React como `useState` y `useEffect` para gestionar el estado y los efectos secundarios.

```jsx
import React, { useEffect, useState } from "react";
import css from "./card.module.scss";
import axios from "axios";
import {
  URL_ESPECIES,
  URL_EVOLUCIONES,
  URL_POKEMON,
} from "../../../api/apiRest";

export default function Card({ card }) {
  const [itemPokemon, setItemPokemon] = useState({});
  const [especiePokemon, setEspeciePokemon] = useState({});
  const [evoluciones, setEvoluciones] = useState([]);

  useEffect(() => {
    const dataPokemon = async () => {
      const api = await axios.get(`${URL_POKEMON}/${card.name}`);
      setItemPokemon(api.data);
    };
    dataPokemon();
  }, [card]);

  useEffect(() => {
    const dataEspecie = async () => {
      const URL = card.url.split("/");
      const api = await axios.get(`${URL_ESPECIES}/${URL[6]}`);
      setEspeciePokemon({
        url_especie: api?.data?.evolution_chain,
        data: api?.data,
      });
    };
    dataEspecie();
  }, [card]);

  useEffect(() => {
    async function getPokemonImagen(id) {
      const response = await axios.get(`${URL_POKEMON}/${id}`);
      return response?.data?.sprites?.other["official-artwork"]?.front_default;
    }

    if (especiePokemon?.url_especie) {
      const obtenerEvoluciones = async () => {
        const arrayEvoluciones = [];
        const URL = especiePokemon?.url_especie?.url.split("/");
        const api = await axios.get(`${URL_EVOLUCIONES}/${URL[6]}`);
        const URL2 = api?.data?.chain?.species?.url?.split("/");
        const img1 = await getPokemonImagen(URL2[6]);
        arrayEvoluciones.push({
          img: img1,
          name: api?.data?.chain?.species?.name,
        });

        if (api?.data?.chain?.evolves_to?.length !== 0) {
          const DATA2 = api?.data?.chain?.evolves_to[0]?.species;
          const ID = DATA2?.url?.split("/");
          const img2 = await getPokemonImagen(ID[6]);
          arrayEvoluciones.push({
            img: img2,
            name: DATA2?.name,
          });

          if (api?.data?.chain.evolves_to[0].evolves_to.length !== 0) {
            const DATA3 =
              api?.data?.chain?.evolves_to[0]?.evolves_to[0]?.species;
            const ID = DATA3?.url?.split("/");
            const img3 = await getPokemonImagen(ID[6]);
            arrayEvoluciones.push({
              img: img3,
              name: DATA3?.name,
            });
          }
        }

        setEvoluciones(arrayEvoluciones);
      };

      obtenerEvoluciones();
    }
  }, [especiePokemon]);

  let pokeId = itemPokemon?.id?.toString();
  if (pokeId?.length === 1) {
    pokeId = "00" + pokeId;
  } else if (pokeId?.length === 2) {
    pokeId = "0" + pokeId;
  }
  return (
    <div className={css.card}>
      <img
        className={css.img_poke}
        src={itemPokemon?.sprites?.other["official-artwork"]?.front_default}
        alt="pokemon"
      />
      <div
        className={`bg-${especiePokemon?.data?.color?.name} ${css.sub_card}  `}
      >
        <strong className={css.id_card}>#{pokeId} </strong>
        <strong className={css.name_card}> {itemPokemon.name} </strong>
        <h4 className={css.altura_poke}> Altura: {itemPokemon.height}0 cm </h4>
        <h4 className={css.peso_poke}>Peso: {itemPokemon.weight} Kg </h4>
        <h4 className={css.habitat_poke}>
          Habitat: {especiePokemon?.data?.habitat?.name}{" "}
        </h4>

        <div className={css.div_stats}>
          {itemPokemon?.stats?.map((sta, index) => {
            return (
              <h6 key={index} className={css.item_stats}>
                <span className={css.name}> {sta.stat.name} </span>
                <progress value={sta.base_stat} max={110}></progress>
                <span className={css.numero}> {sta.base_stat} </span>
              </h6>
            );
          })}
        </div>

        <div className={css.div_type_color}>
          {itemPokemon?.types?.map((ti, index) => {
            return (
              <h6
                key={index}
                className={`color-${ti.type.name}  ${css.color_type} `}
              >
                {" "}
                {ti.type.name}{" "}
              </h6>
            );
          })}
        </div>

        <div className={css.div_evolucion}>
          {evoluciones.map((evo, index) => {
            return (
              <div key={index} className={css.item_evo}>
                <img src={evo.img} alt="evo" className={css.img} />
                <h6> {evo.name} </h6>
              </div>
            );
          })}
        </div>
      </div>
    </div>
  );
}
```

### `LayoutHome.jsx`

Este componente gestiona la paginación, búsqueda y la disposición de las tarjetas de Pokémon en la página principal.

```jsx
import React, { useEffect, useState } from "react";
import css from "./layout.module.scss";
import Header from "../header/Header";
import axios from "axios";
import * as FaIcons from "react-icons/fa";
import { URL_POKEMON } from "../../../api/apiRest";
import Card from "../card/Card";

export default function LayoutHome() {
  const [arrayPokemon, setArrayPokemon] = useState([]);
  const [globalPokemon, setGlobalPokemon] = useState([]);
  const [xpage, setXpage] = useState(1);
  const [search, setSearch] = useState('');

  useEffect(() => {
    const api = async () => {
      const limit = 15;
      const xp = (xpage - 1) * limit;
      const apiPoke = await axios.get(
        `${URL_POKEMON}?offset=${xp}&limit=${limit}`
      );

      setArrayPokemon(apiPoke.data.results);
    };

    api();
    getGlobalPokemons();
  }, [xpage, search]);

  const getGlobalPokemons = async () =>

 {
    const res = await axios.get(`${URL_POKEMON}?offset=0&limit=1000`);

    const promises = res.data.results.map((pokemon) => {
      return pokemon;
    });

    const results = await Promise.all(promises);
    setGlobalPokemon(results);
  };

  const filterPokemons = search?.length > 0
    ? globalPokemon?.filter(pokemon => pokemon?.name?.includes(search))
    : arrayPokemon

  const obtenerSearch = (e) => {
    const texto = e.toLowerCase()
    setSearch(texto)
    setXpage(1)
  }

  return (
    <div className={css.layout}>
      <Header obtenerSearch={obtenerSearch} />

      <section className={css.section_pagination}>
        <div className={css.div_pagination}>
          <span className={css.item_izquierdo}
            onClick={() => {
              if (xpage === 1) {
                return console.log("no puedo retroceder");
              }
              setXpage(xpage - 1);
            }}
          >
            <FaIcons.FaAngleLeft />
          </span>
          <span className={css.item}> {xpage} </span>
          <span className={css.item}> DE </span>
          <span className={css.item}>
            {" "}
            {Math.round(globalPokemon?.length / 15)}{" "}
          </span>
          <span
            className={css.item_derecho}
            onClick={() => {
              if (xpage === 67) {
                return console.log("es el ultimo");
              }
              setXpage(xpage + 1);
            }}
          >
            {" "}
            <FaIcons.FaAngleRight />{" "}
          </span>
        </div>
      </section>

      <div className={css.card_content}>
        {filterPokemons.map((card, index) => {
          return <Card key={index} card={card} />;
        })}
      </div>
    </div>
  );
}
```

## API

La aplicación utiliza la API RESTful de Pokémon para obtener datos sobre los Pokémon, sus especies y sus evoluciones.

### Endpoints Principales

- **`URL_POKEMON`**: Obtiene los datos básicos de los Pokémon.
- **`URL_ESPECIES`**: Obtiene datos sobre las especies de los Pokémon.
- **`URL_EVOLUCIONES`**: Obtiene datos sobre las cadenas de evolución de los Pokémon.

## Contribuciones

Si deseas contribuir a este proyecto, sigue estos pasos:

1. Haz un fork del repositorio.
2. Crea una nueva rama (`git checkout -b feature/nueva-caracteristica`).
3. Realiza tus cambios y haz un commit (`git commit -am 'Añadir nueva característica'`).
4. Envía los cambios a tu rama (`git push origin feature/nueva-caracteristica`).
5. Abre un Pull Request.

## Licencia

Este proyecto está licenciado bajo la Licencia MIT. Consulta el archivo [LICENSE](LICENSE) para obtener más detalles.