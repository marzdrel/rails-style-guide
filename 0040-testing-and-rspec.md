Testing and RS
============

## Table of contents

[...]

## RSpec

- Every newly created file should be created with a corresponding spec file.
When staging changes for commit make sure that a staged file has a corresponding
spec file (if applicable):

```shell
# File with code
app/services/some/example/object.rb

# Corresponding spec file
spec/services/some/example/object_spec.rb
```

Always write spec for your code. Preferably write specs before writing the
implementation. If for some reason you cannot spec the current logic, then at
least create pending spec file. Always include comment with explanation why the
spec is missing.

```ruby
RSpec.describe Some::Example::Object do
  describe "#call" do
    pending __FILE__

    # FIXME I need help with this spec. I do not know how to test this
    # implementation. It uses new framework and there is no examples
    # for this kind of logic in the codebase yet.
  end
end
```

When passing block to single spec use multi-line `do ... end` notation. Always
use double quotes around spec title. Do not use parenthesis after `it` method
name.

Put condition either in spec names. Or wrap the text in `context` block. If
the wording is too long, prefer using `context` block. Dot not use multiple
lines to describe the spec. If you need to provide more context, use multiple
`context` blocks.

```ruby
# Good
describe Contact do
  context "without firstname" do
    it "is invalid" do
      ...
    end
  end
end

# Good
describe Contact do
  it "is invalid without firstname" do
    ...
  end
end
```

In case of very trivial and repeatable specs we do not use titles at all. In
such cases we use one line notation using `it { ... }` syntax. Even though in
general `should` notion is discouraged in the newer versions of RSpec we still
use it for such trivial examples. It reads better and it is much shorter. In
general cases we use one-line notation for various types of ShouldaMatchers.

```ruby
describe Contact do
  it { should validate_presence_of(:firstname) }
  it { should validate_uniqueness_of(:email) }
  it { should validate_length_of(:password).is_at_least(10) }
end
```

- If a spec one-liner exceeds 80 character revert back to a typical multi
line notation. Never omit titles in multi-line notation. If the spec is
still very trivial just use simple `#method` notation.

```ruby
describe "definitions" do
  it "#kind" do
    expect(described_class.new)
      .to define_enum_for(:kind)
      .with(bad: 0, average: 1, good: 2, awesome: 3)
  end

  it "#historic_levels_build_by_day" do
    expect(described_class.new)
      .to delegate_method(:build_by_day)
      .to(:historic_levels)
      .with_prefix
  end

  it "#amount" do
    expect(described_class.new)
      .to have_db_column(:uid)
      .of_type(:string)
      .with_options(null: false)
  end
end
```

### Spec layout and structure (let/before/subject)

Avoid using `let` and `let!` methods and `before` blocks. They make the spec
file harder to read and understand. Instead of using `let` and `let!` methods
place the setup code directly in the spec. Consider moving code to the
`before` block only if the same setup is valid for all specs and there is
no conditional logic involved.

This practice make the specs longer and require lot of duplication, but they
make the logic way easier to follow, change, understand. New/changed specs are
easy to understand even in diffs views, while doing a code review.

Some guides encourage extracting common code to helpers and Ruby methods to
avoid using `let` and `before` blocks. This is a not a good practice. It introduces
the same issues as using `let` and `before` blocks. Just not a solution to core
problem, just a dummy workaround.

In order to be comfortable with this practice you probably need to learn more
advanced editing techniques in your editor. This is a good set of skills anyway.

Please read the following for more information:
- https://thoughtbot.com/blog/lets-not
- https://thoughtbot.com/blog/my-issues-with-let

### Shared examples and logic in specs

Avoid using any kind of shared examples, they are bad form of abstraction and
they quickly become state dependent. They are hard to follow and understand.

Do not auto-generate using loops or any kind of meta programming. If you really
need to iterate a spec over a set of predefined values, consider using the loop
inside the spec, not outside. Use this only if the spec is sizable and splitting
would require a lot of duplication. In any other case simply make multiple specs
or use multiple expectations in a spec.

```ruby
# Bad
["PL", "EN", "DE"].each do |locale|
  it "returns localized title for #{locale}" do
    expect(described_class.call(locale: locale))
      .to eq("...")
  end
end

# Bad
["PL", "EN", "DE"].each do |locale|
  it "returns localized title for locales" do
    expect(described_class.call(locale: locale))
      .to eq("...")
  end
end

# Good
it "returns localized title for locales" do
  expect(described_class.call(locale: "PL"))
    .to eq("...")

  expect(described_class.call(locale: "EN"))
    .to eq("...")

  expect(described_class.call(locale: "DE"))
    .to eq("...")
end

# Good
it "returns localized title for locales PL" do
  expect(described_class.call(locale: "PL"))
    .to eq("...")
end

it "returns localized title for locales EN" do
  expect(described_class.call(locale: "EN"))
    .to eq("...")
end

it "returns localized title for locales DE" do
  expect(described_class.call(locale: "DE"))
    .to eq("...")
end
```

### Invocation of tested method

If the service your are testing has a single class level method, which is a proxy
for the logic of the instance, consider calling this method directly in the spec.

```ruby
# Bad
expect(described_class.new.call).to eq("something")

# Good
expect(described_class.call).to eq("something")
```

