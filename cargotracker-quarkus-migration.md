# Migration from AppServers to Quarkus

```bash
cd cargotracker
```

## Change JakartaEE application to be more compatible with Quarkus and be able to run on JakartaEE servers

- `src/main/java/org/eclipse/cargotracker/application/util/RestConfiguration.java`
    - comment out // import org.glassfish.jersey.server.ServerProperties; and related code

- `src/main/java/org/eclipse/cargotracker/application/internal/DefaultBookingService.java`
    - change //@Stateless to @ApplicationScoped
    - import jakarta.enterprise.context.ApplicationScoped;
    - add import jakarta.transaction.Transactional;
    - add @Transactional to bookNewCargo, assignCargoToRoute, changeDestination, changeDeadline

- `src/main/java/org/eclipse/cargotracker/infrastructure/routing/ExternalRoutingService.java`
    - change //@Stateless to @ApplicationScoped

- `src/main/java/org/eclipse/cargotracker/interfaces/booking/sse/RealtimeCargoTrackingService.java`
    - change // import jakarta.ejb.Singleton; to import jakarta.inject.Singleton;

- `src/main/webapp/public/track.xhtml`
    - replace `public.track` with `publicTrack`

- `src/main/java/org/eclipse/cargotracker/interfaces/tracking/web/Track.java`
    - replace `public.track` with `publicTrack`

- `src/main/webapp/admin/tracking/track.xhtml`
    - replace `admin.track` with `adminTrack`

- `src/main/java/org/eclipse/cargotracker/interfaces/booking/web/Track.java`
    - replace `admin.track` with `adminTrack`

## Copy `src/main/webapp` to the location where Quarkus expects web ui

```bash
mkdir src/main/resources/META-INF/resources
cp -r src/main/webapp/* src/main/resources/META-INF/resources/
cp src/main/resources/META-INF/resources/WEB-INF/web.xml src/main/resources/META-INF/web.xml
```

- remove content after `</welcome-file-list>` from `src/main/resources/META-INF/web.xml`

`cp -r ...` command can be used to sync changes in the future

## Add `src/main/resources/application.properties`

## Create Quarkus implementation for JMS

```bash
cp src/main/java/org/eclipse/cargotracker/infrastructure/messaging/jms/JmsApplicationEvents.java.quarkus src/main/java/org/eclipse/cargotracker/infrastructure/messaging/jms/JmsApplicationEvents.java
cp src/main/java/org/eclipse/cargotracker/infrastructure/messaging/jms/HandlingEventRegistrationAttemptConsumer.java.quarkus src/main/java/org/eclipse/cargotracker/infrastructure/messaging/jms/HandlingEventRegistrationAttemptConsumer.java
```

## Update SampleDataGenerator

```bash
cp src/main/java/org/eclipse/cargotracker/application/util/SampleDataGenerator.java.quarkus src/main/java/org/eclipse/cargotracker/application/util/SampleDataGenerator.java
```

## Start EE AppServers

```bash
scripts/prepare4ee.sh

# Default Payara
mvn clean package cargo:run

# GlassFish
mvn clean package -Pglassfish cargo:run

# OpenLiberty
mvn clean package -Popenliberty liberty:run
```

## Start Quarkus Dev mode

```bash
scripts/prepare4quarkus.sh
export TESTCONTAINERS_RYUK_DISABLED=true
mvn clean package -P quarkus
mvn quarkus:dev -P quarkus
```

## Start Quarkus docker-compose

```bash
scripts/prepare4quarkus.sh
docker build -t cargotracker:latest -f Dockerfile.quarkus --progress=plain .
docker compose up --force-recreate && docker rm $(docker ps -a | grep "cargotracker" | awk '{print $1}')
```

## Start Quarkus native build

```bash
scripts/prepare4quarkus.sh
docker build -t cargotracker:latest -f Dockerfile.native --progress=plain .
docker compose up --force-recreate && docker rm $(docker ps -a | grep "cargotracker" | awk '{print $1}')
```
