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
