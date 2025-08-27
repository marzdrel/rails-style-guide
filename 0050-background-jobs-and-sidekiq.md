Background jobs and Sidekiq
============

## Table of contents

[...]

Do not pass complex object as a parameters to methods which schedule the jobs.
Sidekiq params are serialized and stored in Redis. When the job has to be
done in context of some Record, always pass just the id (number or uuid),
and then load the Object inside worker code.

Do not put any logic inside the typical Sidekiq worker code. Always
encapsulate expected logic into some separate Service Object. Call that
inside `#perform` method. Most prefered way is actually to load expected
object then pass that object to the Service Object.

Write a simple spec for this behavior. Check if the expected object is
loaded using preferred method and if the Service Object is called using
expected method with provided parameteres. Test the core logic in the
Service Object spec outside the worker code.

```ruby
class Agent::Blocker::Worker
  include Sidekiq::Worker
  include RedisMutex::Macro

  auto_mutex :perform, on: [:agent_id]

  def perform(agent_id)
    agent = Agent.find(agent_id)
    Agent::Blocker.call(agent)
  end
end
```
