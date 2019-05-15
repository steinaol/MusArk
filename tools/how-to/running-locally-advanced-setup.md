# Running locally: Advanced setup

Specially when working on the backend, the default `deploy.sh` process described in "[Running system locally](running-system-locally)" can be somewhat cumbersome.

This document tries to remedy some of that by providing an example _custom_ setup that provides a bit more flexibility.

> **¡¡¡IMPORTANT!!!**: If you decide to go down this path you will _have_ to ensure that your custom setup is in sync with the default setup yourself. None of these custom scripts should **ever** be committed to the gitlab repos.

## Step by step instructions

1. Navigate to the `docker/dev` folder.

    ```bash
    cd ~/projects/musit/docker/dev
    ```

2. Create the two new custom files

    ```bash
    # create custom script file
    touch mydeploy.sh

    # make script file executable
    chmod +x mydeploy.sh

    # create custom docker-compose file by copying the default one
    cp docker-compose.yml my-docker-compose.yml
    ```

3. Ensure that the above files are _never_ committed into the git repository (still standing in `docker/dev` folder). Open the following file

    ```bash
    vi ../../.git/info/exclude
    ```

    And add the following two lines before saving and closing:

    ```bash
    docker/dev/mydeploy.sh
    docker/dev/my-docker-compose.yml
    ```

4. Open the `my-docker-compose.yml` file with your favorite editor

    ```bash
    vi my-docker-compose.yml
    ```

    Then **remove** the entire `db` container definition:

   

    And remove all `links` with `db` references. Remove _the entire_  `links` block:

    ```bash
    links:
      - db
    ```

5. Open the `mydeploy.sh` script file

    ```bash
    vi mydeploy.sh
    ```

    Then add the following content, and modify necessary properties:

    ```bash
    echo "################################################################################"
    echo "## IMPORTANT!"
    echo "## This script is ONLY meant for DEVELOPMENT, and NOT for PRODUCTION."
    echo "## If you are seeing this message in PRODUCTION, something has gone terribly wrong!"
    echo "################################################################################"

    ENV=$1
    DOCKER_CMD=docker-compose
    DOCKER_ARGS=""

    # Whenever a new container is added to the docker-compose config, it should be added
    # to the blow string variable.
    DOCKER_IMAGES="dev_backend_1 dev_barcode_1 dev_auth_1 dev_nginx_1"

    MY_IP=$(ipconfig getifaddr en0)

    # ------------------------------------------------------------------------------
    # Slick Database configuration
    # ------------------------------------------------------------------------------
    export EVOLUTION_ENABLED=false
    export APPLICATION_SECRET=dummyAppSecret
    export SLICK_DRIVER=com.typesafe.slick.driver.oracle.OracleDriver$
    export SLICK_DB_DRIVER=oracle.jdbc.OracleDriver
    #:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
    if [ "$ENV" == "--utv" ]; then
      echo "Running local docker services against $ENV"
      DOCKER_ARGS="-f my-docker-compose.yml"
      export SLICK_DB_URL=<INSERT CONNECTION STRING FOR UTV DB HERE>
      export SLICK_DB_USER=<INSERT DB USER HERE>
      export SLICK_DB_PASSWORD=<INSERT PASSWORD HERE>

    elif [ "$ENV" == "--nodb" ]; then
      echo "Running local docker services against DB on $MY_IP"
      DOCKER_ARGS="-f my-docker-compose.yml"
      export SLICK_DB_URL=jdbc:oracle:thin:@$MY_IP:1521:orcl
      export SLICK_DB_USER=musit
      export SLICK_DB_PASSWORD=musit

    else
      echo "Running full docker environment"
      export SLICK_DB_URL=jdbc:oracle:thin:@db:1521:orcl
      export SLICK_DB_USER=musit
      export SLICK_DB_PASSWORD=musit
    fi
    #:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

    # ------------------------------------------------------------------------------
    # The application defaults to use the fake security module
    # (no.uio.musit.security.fake.FakeModule) If you want to test with Dataporten,
    # make sure to remove the comment on the below line(s).
    # Also you will need to set up an application in Dataporten to get access to a
    # Callback URL, Client ID and Client secret.
    # ------------------------------------------------------------------------------
    export MUSIT_SECURITY_MODULE="no.uio.musit.security.dataporten.DataportenModule"
    export CALLBACK_URL="http://musit-test:8888/api/auth/rest/authenticate"
    export CLIENT_ID="ccee5f45-6f32-4315-9a89-9e6ad98a8186"
    export CLIENT_SECRET="d01c882a-b24c-4ebf-9035-381a9a8cd74e"
    export DATAPORTEN_CLIENT_ID=$CLIENT_ID
    export DATAPORTEN_CLIENT_SECRET=$CLIENT_SECRET
    export DATAPORTEN_SESSION_TIMEOUT="4 hours"

    # ------------------------------------------------------------------------------
    # Start the deployment process...
    # ------------------------------------------------------------------------------
    STARTDIR=$(pwd)
    echo "MUSARK: stopping docker images..."
    $DOCKER_CMD $DOCKER_ARGS stop

    echo "MUSARK: removing docker images..."
    docker $DOCKER_ARGS rm -f $DOCKER_IMAGES

    cd ../..
    echo "MUSARK: publishing docker images..."
    sbt docker:publishLocal > /dev/null

    cd ${STARTDIR}
    echo "MUSARK: starting docker environment..."
    $DOCKER_CMD $DOCKER_ARGS up -d --build --force-recreate --remove-orphans
    ```

6. You now have a working, custom, local deployment option. When the above steps are completed, you will be able to execute commands as follows:

    ```bash
    # Run local system using default docker-compose.yml
    ./mydeploy.sh

    # Run local system using my-docker-compose.yml
    ./mydeploy.sh --nodb

    # Run local system using my-docker-compose.yml connecting to UTV database
    ./mydeploy.sh --utv
    ```

# Appendix



### Connecting to UTV DB

To be able to connect to the UTV database, you must set up an SSH tunnel (using sshuttle or similar tools) via the `musit-jump01` machine. Otherwise you will not be able to connect. How to set up an SSH tunnel is not covered here.