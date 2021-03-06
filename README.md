# Sequins ✨

Sequins allows you to define temporal sequences of actions.
A sequence is one or more steps that run in any order you choose with any delay between them you choose.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'sequins'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install sequins

## Usage

### Create a Sequence Class

A basic sequence class looks like this:

```ruby
class ExampleSequence < Sequins::Base
  sequence do
    step :start, initial: true do
      ExampleMailer.example_message(target.id).deliver_later

      delay 3.days, then: :step_2
    end

    step :step2 do
      ExampleMailer.example_step_2(target.id).deliver_later
      end_sequence
    end
  end
end
```

### Steps

A step is a Ruby block. It has the following attributes available:

* `target` is the target object that you pass into `trigger` or `run_step_for_target`
* `step_name` is the step that's currently running
* `sequence` is an instance of `Sequins::Sequence`. From this you have access to the sequence class via `sequence.klass`

You also have direct access to class methods on your `Sequence::Base` subclass. For example:

```ruby
class Example2Sequence < Sequins::Base
  sequence do
    step :start, initial: true do
      send_message target, "hi there"
    end
  end

  def self.send_message(target, text)
    SomeMailer.send_message(target.id, text).deliver_later
  end
end
```

You can also send arguments to `trigger` or `run_step_for_target` which will be passed as block arguments. Example:

```ruby
class Example3Sequence < Sequins::Base
  sequence do
    step :start, initial: true do |something|
      if something.present?
        delay 3.days, then: :next_step
      else
        end_sequence
      end
    end

    step :next_step do
      # something interesting
    end
  end
end

Example3Sequence.trigger(User.first, "this will trigger a delay")
```

### Delay Options

The first argument to `delay` is always an interval, usually expressed as a number of days. Ex: `3.days`.
In addition, it can take these arguments:

* `then` (always required) this tells delay which step to run next
* `at` specifies a time relative to the target's local timezone to run the next step
* `only` limits what days to send on. Currently the only valid option is `:weekdays`

### Trigger a Sequence

To trigger a sequence, call the `trigger` class method and pass in the target. 
For example, if we want to trigger the example sequence for Joel, we'd do this:

```ruby
ExampleSequence.trigger(User.find_by(email: 'joel@egghead.io'))
```

### Hooks

Sequences can define hooks. The available hooks are:

```
before_each_step # runs before every step.
after_each_step  # runs after each step
before_sequence  # runs before the sequence starts. 
after_sequence   # runs after the sequence ends with an explicit `end_sequence`
```

Hooks are run in the same way steps are, so you have access to all of the same attributes as a step. Within `before_each_step` and `after_each_step`, `step_name` will be set to the step that is about to or just finished running. Within `before_sequence` and `after_sequence` `step_name` has no meaning.

## Configuration

```ruby
# in config/initializers/sequins.rb

Sequins.configure do |config|
  # Specify the default time zone in tz format.
  # If you are using Sequins inside Rails this will be set to Rails.configuration.time_zone.
  # You can also change this per-target by providing a `local_time_zone` method on your target.
  config.default_time_zone = 'America/Chicago'
end
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/eggheadio/sequins. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## License

Copyright (c) Peter Keen. The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).

## Code of Conduct

Everyone interacting in the Sequins project’s codebases, issue trackers, chat rooms and mailing lists is expected to follow the [code of conduct](https://github.com/eggheadio/sequins/blob/master/CODE_OF_CONDUCT.md).
