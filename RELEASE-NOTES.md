# Riak 2.1.0 Release Notes

## New Features

### Write Once Bucket Type

Riak 2.1.0 introduces the concept of Write Once Buckets, buckets whose entries are intended to be written exactly once and never updated or overwritten. Since objects are intended to only be written once, Riak does not perform the usual “get, merge, update” cycle which reduces IOPs and improves throughput and latency.

It is still possible for multiple entries to be written to a single key -- through mis-use of the API, network partitions, simultaneous writes, etc.. In these cases, Riak will always resolve siblings using an algorithm based on the SHA-1 hashes of the conflicting objects. While this algorithm is repeatable and deterministic at the database level, it will have the appearance to the user of “random write wins”.

The `write_once` property is a boolean value applied to a bucket type and may only be set at bucket creation time. Once a bucket type has been set and activated with the `write_once` property, the property may not be modified

Limitations:

 * Pre/Post-commit hooks are not supported.
 * Large object warnings and limits are not enforced.
 * All nodes must be upgraded before enabling `write_once` - usage in mixed clusters will crash vnodes.

## Changes

* [Issue kv679](https://github.com/basho/riak_kv/issues/679) - Applies fix to most causes of data loss from repeated causal history. A monotonic counter is fsynced to disk when the vnode starts. Thus starting/re-starting a vnode has a cost associated that was not there before. 
  * [riak_kv/pull/1070](https://github.com/basho/riak_kv/pull/1070)

* Update API to retrieve active preflist based on particular bucket/key.
Updates the Riak API with an operation to GET, e.g. `/types/Type/buckets/Bucket/keys/Key/preflist`, an annotated (nodes being primary or fallback) preflist of which nodes host a particular bucket/key combination is returned.
  * [riak-erlang-http-client/pull/50](https://github.com/basho/riak-erlang-http-client/pull/50)
  * [riak_core/pull/705](https://github.com/basho/riak_core/pull/705)
  * [riak_pb/pull/105](https://github.com/basho/riak_pb/pull/105)
  * [riak-erlang-client/pull/204](https://github.com/basho/riak-erlang-client/pull/204)
  * [riak_kv/pull/1083](https://github.com/basho/riak_kv/pull/1083)
  * [riak_api/pull/81](https://github.com/basho/riak_api/pull/81)
  * [riak_api/pull/75](https://github.com/basho/riak_api/pull/75)

* Add a JSON writer for several riak-admin commands, including cluster status, cluster partition[s], cluster partition-count, any of the riak-admin handoff commands, and riak-admin set/show/describe.
  * [clique/pull/47](https://github.com/basho/clique/pull/47)

* Integrate performance patches into Erlang R16 VM used by Riak.
  * [opt/pull/10](https://github.com/basho/otp/pull/10)

* Add riak_core_ring_manager:is_stable_ring/0 to public API. Allows clients to ensure that the ring has not changed in X seconds (currently 90).
  * [riak_core/pull/716](https://github.com/basho/riak_core/pull/716)

* [yokozuna/issues/468](https://github.com/basho/yokozuna/issues/468) - Add `search_query_latency_mean` and `search_index_latency_mean` metrics.
  * [yokozuna/pull/473](https://github.com/basho/yokozuna/pull/473)

## Fixes

* [yokozuna/issues/437](https://github.com/basho/yokozuna/issues/437): Timeouts during bulk data load, possible tie in to yz_events crash and a bad state on core create.
  * [yokozuna/pull/458](https://github.com/basho/yokozuna/pull/458)

* [yokozuna/issues/452](https://github.com/basho/yokozuna/issues/452):  Don't delete siblings in YZ when using SC or CRDTs.
  * [yokozuna/pull/457](https://github.com/basho/yokozuna/pull/457)

* [yokozuna/issues/402](https://github.com/basho/yokozuna/issues/402): Creating a new search index via HTTP responds before the index is available.
  * [riak_test/pull/745](https://github.com/basho/riak_test/pull/745)
  * [riak_pb/pull/112](https://github.com/basho/riak_pb/pull/112)
  * [riak_pb/pull/111](https://github.com/basho/riak_pb/pull/111)
  * [riak-erlang-client/pull/207](https://github.com/basho/riak-erlang-client/pull/207)
  * [yokozuna/pull/463](https://github.com/basho/yokozuna/pull/463)

* [yokozuna/issues/450](https://github.com/basho/yokozuna/issues/450): AAE Failing hourly, with exit value {badarg,46}, [{base64,decode_binary[{base64.erl…
  * [yokozuna/pull/459](https://github.com/basho/yokozuna/pull/459)

* [yokozuna/issues/437](https://github.com/basho/yokozuna/issues/437): yz_events:handle_info called with bad arguments.
  * [yokozuna/pull/458](https://github.com/basho/yokozuna/pull/458)
  * [yokozuna/pull/42](https://github.com/basho/yokozuna/pull/42)

* [yokozuna/issues/469](https://github.com/basho/yokozuna/issues/469): Fix YZ stats name typo from throughput to throughput.
  * [yokozuna/pull/470](https://github.com/basho/yokozuna/pull/470)

* [riak_core/issues/698](https://github.com/basho/riak_core/issues/698): SSL connections from peers with matching wildcard name are always rejected.
  * [riak_core/pull/718](https://github.com/basho/riak_core/pull/718) 

## Known Issues

* [yokozuna/issues/481](https://github.com/basho/yokozuna/issues/481) - Yokozuna loses entries when yz AAE trees expire. We are currently investigating this issue.


## Notes on upgrading

* Write Once Buckets
  * If you make use of Write Once Buckets, you must upgrade all of your Riak nodes in a cluster to 2.1.0.  There is no support for Write Once Buckets in a cluster with mixed Riak versions.
  * Pre/Post-commit hooks are not supported.
  * Large object warnings and limits are not enforced