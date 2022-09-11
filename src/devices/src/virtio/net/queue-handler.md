Brief description of queue handlers <br />
init: initializing op as EventOps by adding event of a file descriptor, associated data and EVENTSET::IN except TAPFD may be EVENTSET::EDGE_TRIGGERED<br />
process: <br />
  check valid events by matching following values:<br />
    1. events.event_set() == EventSet::IN (| EVENTSET::EDGE_TRIGGERED in case of TAPFD)<br />
    2. events.data() == X_DATA (X may be TAPFD, RX_IOEVENT or TX_IOEVENT)<br />
    3. self.ioeventfd.read().is_err() <br />
    4. inorderhander process_queue() is giving error (queue may be of tap, rx or tx)<br />
  in case of not valid event, remove the event from op.<br />
handle_error: remove all events from ops.<br />
