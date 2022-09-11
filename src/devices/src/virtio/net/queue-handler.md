Brief description of queue handlers
init: initializing op as EventOps by adding event of a file descriptor, associated data and EVENTSET::IN except TAPFD may be EVENTSET::EDGE_TRIGGERED
process: 
  check valid events by matching following values:
    1. events.event_set() == EventSet::IN (| EVENTSET::EDGE_TRIGGERED in case of TAPFD)
    2. events.data() == X_DATA (X may be TAPFD, RX_IOEVENT or TX_IOEVENT)
    3. self.ioeventfd.read().is_err() 
    4. inorderhander process_queue() is giving error (queue may be of tap, rx or tx)
  in case of not valid event, remove the event from op.
handle_error: remove all events from ops.
