Brief description of queue handlers <br/>
init: initializing op as EventOps by adding event of a file descriptor, associated data and EVENTSET::IN except TAPFD may be EVENTSET::EDGE\_TRIGGERED <br/>
process: check valid events by matching following values: <br/>
  1. events.event\_set() == EventSet::IN (| EVENTSET::EDGE\_TRIGGERED in case of TAPFD) 
  2. events.data() == X\_DATA (X may be TAPFD, RX\_IOEVENT or TX\_IOEVENT) 
  3. self.ioeventfd.read().is\_err() 
  4. inorderhander process\_queue() is giving error (queue may be of tap, rx or tx) <br/>
In case of not valid event, remove the event from op. <br/>  
handle\_error: remove all events from ops.
