Brief description of queue handlers  
init: initializing op as EventOps by adding event of a file descriptor, associated data and EVENTSET::IN    
process:  
  check valid events by matching following values:  
      1. events.event\_set() == EventSet::IN  
      2. events.data() == IOEVENT_DATA  
      3. self.ioeventfd.read().is\_err()   
      4. inorderhander process\_queue() is giving error   
  in case of not valid event, remove the event from op.
