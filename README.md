# Google Cloud Firestore Emulator

A [Google Cloud Firestore Emulator](https://cloud.google.com/sdk/gcloud/reference/beta/emulators/firestore/) container image. The image creates a local, standalone Firestore emulator for testing Firestore-backed apps.

## Quickstart

To run the emulator in a standalone Docker container:

```bash
docker run \
  --name firestore-emulator \
  --env "FIRESTORE_PROJECT_ID=dummy-project-id" \
  --env "PORT=8080" \
  --publish 8080:8080 \
  corrdyn/firestore-emulator-docker
```

## Example docker-compose configuration

To run the emulator in a `docker-compose` configuration with your app, use the following example configuration for reference.

```yaml
version: "3.2"
services:
  firestore_emulator:
    image: corrdyn/firestore-emulator
    environment:
      - FIRESTORE_PROJECT_ID=dummy-project-id
      - PORT=8200
  app:
    image: your-app-image
    environment:
      - FIRESTORE_EMULATOR_HOST=firestore_emulator:8200
      - FIRESTORE_PROJECT_ID=dummy-project-id
  depends_on:
    - firestore_emulator
```

## Example GitHub Actions Workflow
Place this in .github/workflows/run_tests.yaml:
```yaml
name: Run Tests
on: push
jobs:
  nox:
    name: Run Tests with Nox
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [ 3.7, 3.8 ]
    services:
      firestore-emulator:
        image: corrdyn/firestore-emulator
        env:
          FIRESTORE_PROJECT_ID: test
          PORT: 8001
        ports:
          # assign random port
          - 8001/tcp
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v2
        with:
          python-version: ${{ matrix.python-version }}
      - name: Install Nox
        run: pip install nox
      - name: Test with Nox
        env:
          # use randomly assigned port
          FIRESTORE_EMULATOR_HOST: localhost:${{ job.services.firestore-emulator.ports[8001] }}
        run: nox
```

## Environment variables

* `FIRESTORE_PROJECT_ID`: This is the Google Cloud Project ID that the emulator uses.
  * This does not have to correspond to a real Google Cloud Project ID, although it can if you want it to.
  * Your application must set its firestore project ID value to match this environment variable.
  * default: `dummy-firestore-id`
* `PORT`: The port on which the firestore emulator listens for HTTP requests.
  * default: `8080`

## Connecting to the emulator

If you're using a standard Firestore client library and set the environment variable `FIRESTORE_EMULATOR_HOST`, then your app will connect to the emulator instead of the production Firestore URL. The variable specifies a host:port pair where the emulator is listening, for example `firestore_emulator:8080`.

Your app must also set the `FIRESTORE_PROJECT_ID` or explicitly set a Firestore project ID to match the `FIRESTORE_PROJECT_ID` you set in the emulator. The image's default project ID is `dummy-firestore-id`.
