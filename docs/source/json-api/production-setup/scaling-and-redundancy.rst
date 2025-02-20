.. Copyright (c) 2022 Digital Asset (Switzerland) GmbH and/or its affiliates. All rights reserved.
.. SPDX-License-Identifier: Apache-2.0

Scaling and Redundancy
######################

.. note:: This section of the document only talks about scaling and redundancy setup for the *HTTP JSON API* server. In all recommendations suggested below we assume that the JSON API is always interacting with a single participant on the ledger.

We advise that the *HTTP JSON API* server and query store components have dedicated
computation and memory resources available to them. This can be achieved via
containerization or by setting them up on independent physical servers. Please ensure that the two
components are **physically co-located** to reduce network latency for
communication. The scaling and availability aspects heavily rely on the interactions between
the core components listed above.

With respect to scaling we recommend one follow general practice: Try to
understand the bottlenecks and see if adding additional processing power/memory helps.

The *HTTP JSON API* can be scaled independently of its query store.
You can have any number of *HTTP JSON API* instances talking to the same query store
(if, for example, your monitoring indicates that the *HTTP JSON API* processing time is the bottleneck),
or have each HTTP JSON API instance talk to its own independent query store
(if the database response times are the bottleneck).

In the latter case, the Daml privacy model ensures that the *HTTP JSON API* requests
are made using the user-provided token, thus the data stored in a given
query store will be specific to the set of parties that have made queries through
that specific query store instance (for a given template).
Therefore, if you do run with separate query stores, it may be useful to route queries
(using a reverse proxy server) based on requesting party (and possibly queried template),
which would minimize the amount of data in each query store as well as the overall
redundancy of said data.

Users may consider running PostgreSQL backend in a `high availability configuration <https://www.postgresql.org/docs/current/high-availability.html>`__.
The benefits of this are use-case dependent as this may be more expensive for
smaller active contract datasets, where re-initializing the cache is cheap and fast.

Finally we recommend using orchestration systems or load balancers which monitor the health of
the service and perform subsequent operations to ensure availability. These systems can use the
`healthcheck endpoints <https://docs.daml.com/json-api/index.html#healthcheck-endpoints>`__
provided by the *HTTP JSON API* server. This can also be tied into supporting an arbitrary
autoscaling implementation in order to ensure a minimum number of *HTTP JSON API* servers on
failures.

Set up the HTTP JSON API Service to work with Highly Available Participants
***************************************************************************

In case the participant node itself is configured to be highly available, depending on the setup you may want
to choose different approaches to connect to the participant nodes. In most setups, including those based on Canton,
you'll likely have an active participant node whose role can be taken over by a passive node in case the currently
active one drops. Just as for the *HTTP JSON API* itself, you can use orchestration systems or load balancers to
monitor the status of the participant nodes and have those point your (possibly highly-available) *HTTP JSON API*
nodes to the active participant node.

To learn how to run and monitor Canton with high availability, refer to the :ref:`Canton documentation <ha_arch>`.

