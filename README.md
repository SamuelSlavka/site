# site

Repo contains infrastructure and keycloak and db form site repositories.

For setup raname `.env.dist` to just `.env`. It will work with default values in dev dicker compose if other projects are also default.

To start using keycloak. After running docker open `http://localhost:8080` then in master dropdown, click add realm. Then import `dev-realm-export.json`. Then create a new user under the new site realm. And set their passwd and roles to at least USER role.

This module has only images so no need to build to run the main proxy use following:

    $ docker-compose up

To run dev enviroment:

    $ docker-compose -f docker-compose-dev.yml up
