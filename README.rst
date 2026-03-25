============================
OpenAds Line Item Manager
============================

Create and manage Google Ad Manager (GAM) line items for OpenAds.

* Free software: Apache Software License 2.0

This is a modified version of `prebid/line-item-manager <https://github.com/prebid/line-item-manager>`_.
The README and documentation on the prebid website can help with any questions that this README doesn't cover.


Overview
--------

This tool automates the creation of line items in Google Ad Manager for publishers
using OpenAds. It supports two modes:

- **Send Top Bid** (``-s`` / ``--single-order``): Creates a single set of orders
  using the top bid across all bidders. Targeting keys use the base ``oa_`` prefix
  (e.g., ``oa_pb``).

- **Send All Bids** (``-b thetradedesk``): Creates separate orders and line items
  per bidder. Targeting keys include the bidder code suffix
  (e.g., ``oa_pb_thetradedesk``). Note: GAM has a 20 character limit, so some may be trunated - that's okay.

Prerequisites
-------------

1. A Google Ad Manager account with API access enabled
2. A JSON private key file for a GAM service account

To set up GAM API access:

1. In `Google API Console <https://console.developers.google.com/>`_ generate a
   private key file for a service account.
2. In Google Ad Manager, enable API access and create a new services user with
   Administrator role. See `detailed instructions
   <https://developers.google.com/ad-manager/api/authentication#oauth>`_.
3. Save the JSON key file as ``gam_creds.json`` (or any name you prefer).

Quick Start
-----------

The easiest way to run this tool is using Docker, though you could do it via pip directly if you don't have docker. The details for that are at the bottom of this file.

We've created 2 basic examples for you to follow - `examples/config_all_bids.yml` and `examples/config_top_bid.yml`, with the universal creative and vast xml caching path already filled out. If those fit your needs (which it will for most), you can follow the remaining directions to copy, modify, and run those.
Otherwise, you can 

1. Build and start the Docker container, mounting the repo into it
::

   # Linux / Mac
   $ extra_run_opts="-v $(pwd):/home/app" command="bash" make docker-run

   # Windows (PowerShell)
   $ $env:extra_run_opts="-v ${PWD}:/home/app"; $env:command="bash"; make docker-run

   # Windows (CMD)
   $ set extra_run_opts=-v %cd%:/home/app && set command=bash && make docker-run

2. Inside the container, copy an example config and edit it
::

   # For Send Top Bid mode:
   $ cp examples/config_top_bid.yml my_config.yml

   # Or for Send All Bids mode:
   $ cp examples/config_all_bids.yml my_config.yml

3. Edit ``my_config.yml`` — at minimum, you'll need to set the ``network_code``, ``network_name``, advertiser name, creative sizes, and rate - though there are a lot of additional config options.

4. Place your GAM credentials JSON file in the repo directory (it will be available inside the container via the mount). See Prerequisites above.

5. Do a dry run to verify everything looks right (no line items are created)
::

   # Send Top Bid mode
   $ line_item_manager create my_config.yml -k gam_creds.json -s --dry-run -v

   # Send All Bids mode
   $ line_item_manager create my_config.yml -k gam_creds.json -b thetradedesk --dry-run -v

6. (Optional) Do a test run creating a limited number of line items for visual inspection in GAM. You should then delete these before running the rest.
::

   # Send Top Bid mode
   $ line_item_manager create my_config.yml -k gam_creds.json -s --test-run -v

   # Send All Bids mode
   $ line_item_manager create my_config.yml -k gam_creds.json -b thetradedesk --test-run -v

7. Create all line items
::

   # Send Top Bid mode
   $ line_item_manager create my_config.yml -k gam_creds.json -s

   # Send All Bids mode
   $ line_item_manager create my_config.yml -k gam_creds.json -b thetradedesk

Running Without Docker
~~~~~~~~~~~~~~~~~~~~~~

If you prefer not to use Docker, you can install directly (using a virtual environment
is recommended)::

   $ python3 -m venv .venv
   $ source .venv/bin/activate
   $ pip install -e .

Configuration
-------------

The example configs in ``examples/`` are the easiest starting point. They include
comments indicating which fields you need to change.

Key sections in the config file:

- **publisher**: Your GAM network code and name (can also be passed via CLI flags
  ``--network-code`` and ``--network-name``)
- **advertiser**: Defaults to "OpenAds"
- **creative**: Banner sizes, snippet, and/or video sizes with VAST URL
- **order / line_item**: Naming templates and line item type
- **targeting**: Optional custom key-value targeting (country, site, etc.)
- **rate**: Currency and price granularity

To see the full default config with all options:
::

   $ line_item_manager show config

Advanced Features
-----------------

1. Use a custom line item template
::

   $ line_item_manager show template > my_template.yml
   # edit my_template.yml, then:
   $ line_item_manager create my_config.yml -s --template my_template.yml

2. Use a custom settings file
::

   $ line_item_manager show settings > my_settings.yml
   # edit my_settings.yml, then:
   $ line_item_manager create my_config.yml -s --settings my_settings.yml

Local Development
-----------------

Using Docker::

   $ git clone <repo-url>
   $ cd line-item-manager
   $ extra_run_opts="-v $(pwd):/home/app" command="bash" make docker-run
   # Inside the container:
   $ make test

Or without Docker::

   $ git clone <repo-url>
   $ cd line-item-manager
   $ python3 -m venv .venv && source .venv/bin/activate
   $ pip install -e .[test]
   $ make test

See ``make help`` for all available Makefile targets.
