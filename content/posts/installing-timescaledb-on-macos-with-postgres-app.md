+++
title = "Installing TimescaleDB on macOS with Postgres.app"
description = "A step-by-step guide to installing TimescaleDB on macOS if you're using Postgres.app."
date = 2025-09-15
updated = 2025-09-15
draft = false
canonical_url = "https://medium.com/@ekachai.suriya_61801/installing-timescaledb-on-macos-with-postgres-app-a-complete-guide-e0f29b14a2be"

[extra]
lang = "en"
toc = true
comment = false
copy = false

[taxonomies]
tags = ["tutorial", "database", "timescaledb"]
+++

## Why the "Easy Way" Didn't Work

If you are a macOS user like me, you probably love using [Postgres.app](https://postgresapp.com/). It's just the easiest way to manage PostgreSQL on a Mac. But recently, I ran into a bit of a headache when I tried to add **TimescaleDB** to the mix.

I started by following the official docs and tried the Homebrew method. "Easy peasy," I thought. But nope. It turns out that Homebrew's version of TimescaleDB is built for Homebrew's PostgreSQL, not the one from Postgres.app. This led to all sorts of annoying issues:

1.  **Target Mismatch**: The extension was compiled for the wrong Postgres version.
2.  **Missing Files**: The `.so` library files weren't where Postgres.app looked for them.
3.  **Path Nightmares**: I had multiple Postgres installations fighting each other.

After banging my head against the wall for a bit (and seeing a lot of empty directories where files *should* have been), I realized that **compiling from source** was actually the way to go.

## Why Building from Source is Better Here

When we build it ourselves, we can tell the build process *exactly* where our Postgres.app lives. This means:

*   It uses the correct `pg_config` from Postgres.app.
*   It puts the library files right where they need to be.
*   Everything is compatible with the specific version of Postgres you are running.

So, let's get to it! Here is the step-by-step guide I wish I had when I started.

## Step-by-Step Installation

### Prerequisites

First off, make sure your Postgres.app is actually running. We also need to make sure your terminal knows where to find the Postgres tools.

Run this to check:

```bash
which pg_config
# You want to see something like: /Applications/Postgres.app/Contents/Versions/latest/bin/pg_config
```

If it shows a different path (like `/usr/local/bin/...`), you need to update your path. Add this to your `~/.zshrc` or `~/.bash_profile`:

```bash
export PATH="/Applications/Postgres.app/Contents/Versions/latest/bin:$PATH"
```

Don't forget to restart your terminal or run `source ~/.zshrc` after saving!

### Step 1: Clean Slate (Optional)

If you already tried the Homebrew method and it failed, let's clean that up first so we don't have conflicts.

```bash
brew uninstall timescaledb
brew untap timescale/tap
```

### Step 2: Get the Build Tools

We need `cmake` to build the project. If you don't have it yet:

```bash
brew install cmake
```

### Step 3: Download TimescaleDB

Let's grab the code from GitHub.

```bash
# Clone the repo
git clone https://github.com/timescale/timescaledb.git

cd timescaledb

# I'm using version 2.21.1 here, but feel free to check out the latest stable tag
git checkout 2.21.1
```

### Step 4: Configure the Build

This is the important part. We need to configure the build so it knows about our Postgres.app environment.

```bash
OPENSSL_ROOT_DIR=/opt/homebrew/opt/openssl ./bootstrap -DREGRESS_CHECKS=OFF
```

**Note for Intel Mac users:** If the path above doesn't work, your OpenSSL might be in a different spot. Try this instead:

```bash
OPENSSL_ROOT_DIR=/usr/local/opt/openssl ./bootstrap -DREGRESS_CHECKS=OFF
```

*Quick tip: We use `-DREGRESS_CHECKS=OFF` because the regression tests need some extra binaries that we don't strictly need just to get the extension running.*

### Step 5: Compile and Install

Time to build! This might take a few minutes, so maybe grab a coffee ☕.

```bash
cd ./build && make
sudo make install
```

When you run `sudo make install`, it will copy the compiled files into your Postgres.app application bundle. This is exactly what we want!

### Step 6: Tell Postgres About It

Now we need to enable the library in your Postgres config.

Find your config file:

```bash
psql -d postgres -c "SHOW config_file;"
```

Open that file in VS Code (or Vim if you're feeling brave) and look for `shared_preload_libraries`. Change it to:

```ini
shared_preload_libraries = 'timescaledb'
```

### Step 7: Restart and Enable

1.  **Important:** Quit Postgres.app completely (Command+Q) and open it again. Just restarting the server isn't enough to load the new library.
2.  Connect to your database and create the extension:

```sql
CREATE EXTENSION IF NOT EXISTS timescaledb;
```

If you see the welcome message, congratulations! You did it! 🎉

### Step 8: Let's Test It

Just to be 100% sure, let's make a quick hypertable.

```sql
-- 1. Create a regular table
CREATE TABLE conditions (
    time TIMESTAMPTZ NOT NULL,
    location TEXT NOT NULL,
    temperature DOUBLE PRECISION NULL
);

-- 2. Turn it into a hypertable
SELECT create_hypertable('conditions', 'time');

-- 3. Add some data
INSERT INTO conditions VALUES 
    (NOW(), 'Bangkok', 32.5),
    (NOW() - INTERVAL '1 hour', 'Chiang Mai', 28.2);

-- 4. Check if it's there
SELECT * FROM conditions ORDER BY time DESC;
```

Hope this guide saved you some time! Happy coding! 🚀
