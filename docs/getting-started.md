# Getting Started

## Step 1 - Run the System
Before proceeding, install the following prerequisites:

- [Docker](https://docs.docker.com/install/) (version 18.09)
- [Docker compose](https://docs.docker.com/compose/install/) (version 1.24.1)

Once everything is installed, execute the following command from project root:

```bash
make run
```

This will start Mainflux docker composition, which will output the logs from the containers.

## Step 2 - Install the CLI
Open a new terminal from which you can interact with the running Mainflux system. The easiest way to do this is by using the Mainflux CLI,
which can be downloaded as a tarball from GitHub (here we use release `0.13.0` but be sure to use the [latest CLI release](https://github.com/mainflux/mainflux/releases)):

```bash
wget -O- https://github.com/mainflux/mainflux/releases/download/0.13.0/mainflux-cli_0.13.0_linux-amd64.tar.gz | tar xvz -C $GOBIN
```

> Make sure that `$GOBIN` is added to your `$PATH` so that `mainflux-cli` command can be accessible system-wide

#### Build mainflux-cli
Build `mainflux-cli` if the pre-built CLI is not compatible with your OS, i.e MacOS. Please see the [CLI](cli.md) for further details.

## Step 3 - Provision the System
Once installed, you can use the CLI to quick-provision the system for testing:
```bash
mainflux-cli provision test
```

This command actually creates a temporary testing user, logs it in, then creates two things and two channels on behalf of this user.
This quickly provisions a Mainflux system with one simple testing scenario.

You can read more about system provisioning in the dedicated [Provisioning](/provision/#platform-management) chapter

Output of the command follows this pattern:

```json
{
  "email": "friendly_beaver@email.com",
  "password": "12345678"
}


"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NDcwMjE3ODAsImlhdCI6MTU0Njk4NTc4MCwiaXNzIjoibWFpbmZsdXgiLCJzdWIiOiJmcmllbmRseV9iZWF2ZXJAZW1haWwuY29tIn0.Tyk31Ae680KqMrDqP895PRZg_GUytLE0IMIR_o3oO7o"


[
  {
    "id": "513d02d2-16c1-4f23-98be-9e12f8fee898",
    "key": "69590b3a-9d76-4baa-adae-9b5fec0ea14f",
    "name": "d0",
  },
  {
    "id": "bf78ca98-2fef-4cfc-9f26-e02da5ecdf67",
    "key": "840c1ea1-2e8d-4809-a6d3-3433a5c489d2",
    "name": "d1",
  }
]


[
  {
    "id": "b7bfc4b6-c18d-47c5-b343-98235c5acc19",
    "name": "c0"
  },
  {
    "id": "378678cd-891b-4a39-b026-869938783f54",
    "name": "c1"
  }
]
```

In the Mainflux system terminal (where docker compose is running) you should see following logs:
```bash
mainflux-users  | {"level":"info","message":"Method register for user friendly_beaver@email.com took 97.573974ms to complete without errors.","ts":"2019-01-08T22:16:20.745989495Z"}
mainflux-users  | {"level":"info","message":"Method login for user friendly_beaver@email.com took 69.308406ms to complete without errors.","ts":"2019-01-08T22:16:20.820610461Z"}
mainflux-users  | {"level":"info","message":"Method identity for client friendly_beaver@email.com took 50.903µs to complete without errors.","ts":"2019-01-08T22:16:20.822208948Z"}
mainflux-things | {"level":"info","message":"Method add_thing for token eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NDcwMjE3ODAsImlhdCI6MTU0Njk4NTc4MCwiaXNzIjoibWFpbmZsdXgiLCJzdWIiOiJmcmllbmRseV9iZWF2ZXJAZW1haWwuY29tIn0.Tyk31Ae680KqMrDqP895PRZg_GUytLE0IMIR_o3oO7o and thing 513d02d2-16c1-4f23-98be-9e12f8fee898 took 4.865299ms to complete without errors.","ts":"2019-01-08T22:16:20.826786175Z"}

...

```

This proves that these provisioning commands were sent from the CLI to the Mainflux system.

## Step 4 - Send Messages
Once system is provisioned, a `thing` can start sending messages on a `channel`:

```bash
mainflux-cli messages send <channel_id> '[{"bn":"some-base-name:","bt":1.276020076001e+09, "bu":"A","bver":5, "n":"voltage","u":"V","v":120.1}, {"n":"current","t":-5,"v":1.2}, {"n":"current","t":-4,"v":1.3}]' <thing_key>
```

For example:
```bash
mainflux-cli messages send b7bfc4b6-c18d-47c5-b343-98235c5acc19 '[{"bn":"some-base-name:","bt":1.276020076001e+09, "bu":"A","bver":5, "n":"voltage","u":"V","v":120.1}, {"n":"current","t":-5,"v":1.2}, {"n":"current","t":-4,"v":1.3}]' 69590b3a-9d76-4baa-adae-9b5fec0ea14f
```

In the Mainflux system terminal you should see following logs:

```bash
mainflux-things | {"level":"info","message":"Method can_access for channel b7bfc4b6-c18d-47c5-b343-98235c5acc19 and thing 513d02d2-16c1-4f23-98be-9e12f8fee898 took 1.410194ms to complete without errors.","ts":"2019-01-08T22:19:30.148097648Z"}
mainflux-http   | {"level":"info","message":"Method publish took 336.685µs to complete without errors.","ts":"2019-01-08T22:19:30.148689601Z"}
```

This proves that messages have been correctly sent through the system via the protocol adapter (`mainflux-http`).
