SCOMERGER
=========

`scomerger` is a seiscomp3 application which aims to merge several origins
belonging to the same event from differents agencys and authors. A new origin
is created with arrivals of all origins and then a relocation (LocSAT with
iaspei91 profile by default) is done.

`scomerger` works like this:

- it listens to `EVENT` messages from `scevent`,
- when an `EVENT` message is received :

    - if a merged origin exists without new origin, nothing is done.
    - if a merged origin exists but a newer origin exists, `scomerger` send a
	  message to `scevent` to remove this merged origin. `scevent` selects
      then a new preferred origin and sends a new `EVENT` message (which will
      be received and processed by `scomerger`).
	- If no merged origin is present, select all the origins that match the
      input agencys, authors and evaluation mode. Only the newest origin is
      kept for a given agency. So, `scomerger` merges these origins and a new
      origin is created with all arrivals and relocated (`LOCSAT` by default,
      magnitude is not yet updated for the moment).
    - `scomerger` send a message to `scevent` to create the new origin in the
	  database. An `originreference` is created as well to associate the origin
	  to the event. `scevent` selects also a new preferred origin.

> **Note**
>
> The merged origin is detected with `author` (**scomerger** by default) and
> `agencyID` fields.
>
> `scomerger` can send the messages to 2 differents `scevent`. The actions of
> removing merged origins and merging can indeed be separated (parameters
> `scomerger/connection/groupRemove` and `global/connection/primaryGroup`).
>
> Every modifications of the database must be done by `scevent`. That's why
> `scomerger` send messages to it.

![Schéma](docs/schema.png)

Installation
------------

1. Copy the different files into seiscomp3 folders:

    ```bash
    cp descriptions/scomerger.xml /path/to/seiscomp3/etc/descriptions/
cp defaults/scomerger.cfg /path/to/seiscomp3/etc/defaults/
cp init/scomerger.py /path/to/seiscomp3/etc/init/
cp scomerger /path/to/seiscomp3/bin/
    ```

2. Configure `scomerger`

3. Activate the seiscomp3 module:

    ```bash
    seiscomp enable scomerger
seiscomp start scomerger
    ```

Useful options
--------------

`scomerger` can be run on a particular event with the `-E` or `--event` option:

```bash
seiscomp exec scomerger -E eost2016ehix
```

It can be run on a period of time as well:

```bash
seiscomp exec scomerger --begin 2016-03-01T00:00:00.0 --end 2016-03-08T00:00:00.0
```

It is also possible to remove the merged origins with the `-R` option:

```bash
seiscomp exec scomerger -E eost2016ehix -R
```
