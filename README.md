Style Guides
============

Guides for programming in good, consistent style

Git commits
-------------------

- Commit small changes often instead of doing big commits every now
  and then
- Do not mix different issues in single commit
- If you spot any minor style issues while working on a feature commit
  them separately
- Always review changes before commiting to avoid accidental mistakes,
  and/or to avoid random files beeing added to the repository
- Do not rush your commits, review file changes and commit messages
  carefully before submitting

Git commit messages
-------------------

    Capitalized, short (50 chars or less) summary

    More detailed explanatory text, if necessary. Wrap it to about 72
    characters.

    - If there is a need to explain the changes further use bullet
      points with dashes
    - Wrap the points at 72 characters too
    - Separate paragraphs with new line
    - Do not use dots in the title or bullet points
    - Write your commit message in the imperative (Fix bug, not Fixed
      bug, Fixes the bug)
    - Valid title should nicely end this sentence: "If submitted this
      commit will: Fix bug"

    http://project.management-system.com/ticket/123

Details:

http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html
https://github.com/thoughtbot/guides/blob/master/protocol/git/README.md

Method Objects
-------------------

Encapsulate application logic inside Method Objects. Do not relay on class
methods to provide functionality. Work on class instances instead. In order to
simply API and testing always provide a shortcut/proxy class method.

It sole purpose should be to instantiate the class with given arguments and
then call the main instance method. In general case the main method should be
called #call.

Always make internal methods private. Ideal Method Object implementation should
have only one public method.

Even if the logic is extremely simple and it takes only one line, do not
use class methods to provide implementation. Create snippets and marcos in your
development evironment to make the process of createing new classes more easy
and frictionless.

    # Divide two numbers or return zero if the second argument is
    # equal to zero.

    class Divider
      def self.call(*args)
        new(*args).call
      end

      def initialize(arg1, arg2)
        self.arg1 = arg1
        self.arg2 = arg2
      end

      def call
        return 0 if arg2.zero?
        arg1 / arg2
      end

      private

      attr_accessor :arg1, :arg2
    end

If instead of return value you need to work on class instance just modify the
proxy class method to execute the logic and then return an instance instead.
This approach shouldn't be used often outside of the facade / action objects
used in controllers.

      def self.call(*args)
        instance = new(*args)
        instance.call
        instance
      end

Method Objects Specs
--------------------

Always create spec files for every method object. If for some reason you can't
provide spec for given method object at the time create pending spec file with
the corresponding path.

    RSpec.describe Divider do
      describe ".call" do
        subject { described_class }

        it { should forward_to_instance(:call).with_1_args }
      end

      describe "#call" do
        let(:service) { described_class.new(10, arg) }

        context "with non-zero argument" do
          let(:arg) { 5 }

          it "returns division result" do
            expect(service.all).to eq 2
          end
        end

        context "with zero argument" do
          let(:arg) { 0 }

          it "returns 0" do
            expect(service.call).to eq 0
          end
        end
      end
    end

If the logic provides vastly different paths based on the input always create
corresponding branches using contexts (zero / non zero input in the Divider
example). Use common sense for amount of context. In general use border
values and some middle values when dealing with range arguments.

Always provide isloated envorinment for method object specs. If there
are calls to external objects inside a method object - stub the calls with
resonable values. Always check if the calls was actually made in expected
conditions and check if the expected value was passed to the call.

Preffer stubs instead of dependency injection. Due to nature of Ruby semantics
heavy dependency injection makes the code hard to reason about. Verified
stubs also provide basic consistency checks without resorting to integration
testing.
