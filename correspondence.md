pubsub -> modeled: broadcast semantics, subscribers tasks; abstracted: network I/O, StreamMap replaced by select; notes: focus on lag/drop and subscribe/publish races

background-expiration -> modeled: shared Arc<Mutex<State>> + purge task + Notify; abstracted: BTree internals; notes: test notify ordering and expiration insert/remove races
