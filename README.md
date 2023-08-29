Role Name
=========

Role to install and configure clockwork.

Requirements
------------

This was only tested with ansible 2.12.

Role Variables
--------------

Here is a list of the important variables. Those marked with [SECRET]
contain private information that should be kept secure in a vault or
something similar.

 - `clockwork_server_name`: The fqdn for the clockwork web server.
 - `clockwork_google_client_id`: The Google client ID for OAuth.
 - `clockwork_google_client_secret`: [SECRET] The Google client secret for OAuth.
 - `clockwork_ssh_private_key`: [SECRET] SSH private key to connect to remote clusters
 - `clockwork_ssl_cert_file`: [SECRET] SSL certificate file for the website
 - `clockwork_ssl_cert_key`: [SECRET] SSL certificate key for the website
 - `clockwork_ldap_certificate`: [SECRET] SSL certificate for the ldap server (only useful for user sync)
 - `clockwork_ldap_private_key`: [SECRET] SSL private key for the ldap server (only useful for user sync)
 - `clockwork_secret_key`: [SECRET] Flask secret key used to secure session cookies
 - `clockwork_logging_level`: level of verbosity for logging
 - `clockwork_settings_language`: default language for users ("en" or "fr")
 - `clockwork_sentry_dns`: Integrate with sentry for error reporting
 - `clockwork_sentry_traces_sample_rate`: sample rate for sentry traces
 - `clockwork_clusters`: A list of mappings that describe the monitored clusters.
    Here is an example with a description of the fields:
    ```
      # The name is a user-friendly name to identify this entry.  Should not contain spaces
    - name: cluster

      # This is the name of a MongoDB field that will be used to store the account name for each user on this cluster.
      # Multiple cluters can have the same field to share username
      # This must avoid clashing with builtin clockwork fields.
      account_field: cluster_username

      # This is the name of a MongoDB field that will be used to store the account update key for each user on this cluster.
      # If clusters share the account_field, they should also share the update_field.
      # If clusters share the update_field, but not the account_field, only one of them will be updated.
      # If you don't want user updates, set it to false.
      # This must avoid clashing with builtin clockwork fields.
      update_field: cluster_update_key

      # The allocations to monitor, as a list.
      # Can be set to "*" to monitor all allocations
      allocations: [ alloc1, alloc2, alloc4 ]
      allocations: "*"

      # The timezone for the cluster
      # This is used in the import scripts since slurm report times in local time with no indications of the timezone.
      timezone: America/Montreal

      # The organization responsible for the cluster, optional
      organization: "Mila"

      # Some information about the cluster, all optional
      nbr_cpus: 200
      nbr_gpus: 50
      official_documentation: "https://cluster.example.com/docs/"
      mila_documentation: "https://mila.example.com/docs/cluster/"

      # Sorting order for clusters, lower value is higher in display, optional
      display_order: 1

      # Set to false to disable remote fetching of stats.
      fetch_remote: true

      # Connection information for remote fetching
      # Optional if fetch_remote is set to false
      # will run rsync {remote_user}@{remote_hostname}:{remote_path} /some/local/path
      remote_user:
      remote_hostname:
      remote_path:

      # Port used for remote fetching
      ssh_port: 22

      # The interval at which to fetch stats, in minutes.
      # Must be one of 1, 2, 3, 4, 5, 6, 10, 12, 15, 20, 30
      remote_interval: 10

      # Use sacct to get better stats
      # This also requires that remote_user and remote_hostname are
      # properly set
      sacct_enabled: false

      # Mandatory if sacct_enabled is true
      sacct_path: "/opt/software/slurm/bin/sacct"
      
      # Path of sinfo binary on the remote host
      sinfo_path: "/opt/software/slurm/bin/sinfo"
      
      # Name of the SSH key used for remote fetching
      ssh_key_filename: id_ssh 
    ```
  - `clockwork_mongodb_connection_string`: If using the included mongod role, the default is okay.  If using an extrenal mongodb server then it should be set to match.  [SECRET] if using username and password.

There are other variables to change the local username and other minor
details, refer to the defaults/main.yml file for more information.

Example Playbook
----------------

This is a sample that uses all the defined tasks:

   - hosts: clockwork-web
       tasks:
        - name: Setup clockwork server
          include_role:
            name: mila.clockwork
            tasks_from: setup_server
   - hosts: clockwork-scrape
       tasks:
        - name: Setup clockwork stats collection
          include_role:
            name: mila.clockwork
            tasks_from: setup_scrape
        # This step is optional and you can have another way to insert users
        # If included it MUST come after the stats collection and be on
        # the same machine
        - name: Setup user synchcronization with google ldap
          include_role:
            name: mila.clockwork
            tasks_from: setup_users

The setup_user step has many hardcoded choices for now which might not
be useful everywhere, instead of running it as-is, you might want to
adjust it.

Also, for development purposes a setup_mongod step is included, but it
requires that all the processes are on the same machine. It is
strongly recommemed to not deploy in production in that state.

License
-------

MIT
