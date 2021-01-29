Docker
=======


Commands
--------

Turn off all containers

.. code-block:: bash

    docker container stop $(docker container ls -aq)

using a filter

.. code-block:: bash

    docker container prune --filter "until=12h"

`A great resource on explaining the docker basics <https://stackoverflow.com/questions/35000484/how-to-tag-a-docker-container>`_


Docker build

.. code-block:: bash

    DOCKER_BUILDKIT=1 docker build -t ciaranwelsh/libomexmeta-build:latest .

Push to dockerhub

.. code-block:: bash

    docker push ciaranwelsh/libomexmeta-build:latest