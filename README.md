# APEX Manager - command-line multi-launcher + auto-rejoin (Windows)

Launches several Roblox accounts into one game and keeps them there: if a client
closes, crashes, gets kicked, or loses internet, it is relaunched.

## Setup
    pip install -r requirements.txt
    python main.py            (or double-click ram.bat)

Copy your old `data/accounts.json` into `data/` - the format is unchanged.

## Build a Windows executable
Install the pinned dependencies and build the standalone console executable:
    python -m pip install -r requirements.txt
    build.bat

The executable is created at `dist/APEXManager.exe`. Its `data/` directory is created
beside the executable on first launch. Account data is not bundled into the executable.
For a public beta, distribute only the executable and sanitized documentation/settings.
Never distribute `accounts.json`, `.salt`, `browser_profiles`, build folders, or a full
development checkout. Existing cookies should be revoked if they were ever committed
or copied into an exposed repository/archive.

## Quick start
    python main.py add                     # paste cookie (auto-cleaned of hidden characters) + validated
    python main.py run -p 1234567890       # place id or roblox.com/games/... link
    python main.py run                     # next time: place + interval are remembered

## Commands
    run [-p PLACE] [-j JOBID_OR_SERVER_LINK] [-i SECONDS] [-d DELAY] [-g GRACE]
        [-a acct ...] [--kill-existing] [--kill-on-exit] [--no-multi]
    add [user] [cookie] | remove NAME | list | check | import FILE
    set [key value]        interval (default 2), launch_delay, grace, misses,
        net_grace, place_id, job_id, roblox_path, multi_instance, kill_on_exit

    Interface layout (menu option 9 lists every setting with its current value)
        pad_banner   left padding of the ASCII art      -1 = centre (default)
        pad_menu     left padding of the menu buttons   -1 = centre (default)
        pad_prompt   left padding of the "> " input     -1 = centre (default)
        pad_log      left padding of the log lines      -1 = centre (default)
                     (the Start-screen table is always plain left aligned)
        pad_status   left padding of the place line     -1 = centre (default)
        console_width / console_height
                      locked terminal size in cells; 0 = keep the default (80x24)

    Examples
        python main.py set pad_menu 0            menu flush to the left
        python main.py set pad_banner 6          indent the ASCII art
        python main.py set console_width 100     wider terminal
    Interactive menu option 12 adds a Job ID from a Roblox server link
    passwd [--remove]      encrypt accounts.json (env RAM_PASSWORD for unattended runs)
    kill                   close all Roblox clients
    unlock                 test the multi-instance unlock on running clients

## How rejoin works (every `interval` seconds, default 2)
1. Tracked client process gone -> relaunch immediately.
2. Own presence (Roblox API) is not "in game" in your PlaceId for `misses` checks in a row
   (default 2) -> close that client and relaunch. Covers kicks/disconnect screens.
3. Internet down -> checks pause; when it returns, clients get `net_grace` seconds to
   reconnect themselves before being relaunched.
4. After each launch there is a `grace` period (default 8s) before presence is trusted,
   because presence lags behind joining.
5. Failed launches back off 10s, 20s, 40s, 60s. Expired cookies disable that account.

Notes: run as Administrator if `unlock` reports it cannot open a process.
Never share `data/accounts.json` - it contains login cookies.

## Troubleshooting
`python main.py diag` tests your connection to Roblox and checks your cookie for hidden characters.
