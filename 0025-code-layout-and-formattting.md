Style Guides
============

## General formatting rules

The main goal of this guide is to make the code more readable and maintainable.
Bear in mind, that while readability might be subjective, there are some common,
well documented practices that can help in general clarity of the code. Even if
you feel uncomfortable with some of the rules, try to understand the underlying
reason. Very often, the only thing making the code "hard to read" is nostalgia
and familiarity with the previous style.

While consistency helps in readability, there is little value in keeping
some existing, outdated style just for the sake of it. We can't easily update
the layout of the whole codebase, so some discrepancies are expected.

Some of the most crucial factors when it comes to deciding on the style are:
- Diff churn. How much of the code will have to change, if we will decide on
  making a small change?
- Ease of reading. How easy is it to read to follow the code? Is skimming easy?
  Typing should be secondary to reading. Making the quick to write and condensed
  makes it harder to read.

## Deep indent

Avoid using deep indent while writing code. In general deep-indent is any block
which is indented more than 1 level deep, than the surrounding code.

Examples of deep indent:

```ruby
# Example 1
output = if condition
           "output1"
         else
           "output2"
         end

# Example 2
scope = Order
        .joins(:shipment)
        .where(shipment: { status: "shipped" })

# Example 3
User.active
    .joins(:orders)

# Example 4
has_many :vms_agreements,
         -> { order(created_at: :desc) },
         class_name: "::VmsAgreement",
         dependent: :destroy
```

Examples without deep indent:

```ruby
# Example 1
output =
 if condition
   "output1"
 else
   "output2"
 end

# Example 2
scope =
  Order
  .joins(:shipment)
  .where(shipment: { status: "shipped" })

# Example 3
User
  .active
  .joins(:orders)

# Example 4
has_many(
  :vms_agreements,
  -> { order(created_at: :desc) },
  class_name: "::VmsAgreement",
  dependent: :destroy,
)
```

While deep indent might seem like a good visual cue, it makes it harder to read
and follow the logic. If you have to use deep indent, consider refactoring the
code into smaller methods or objects.

Deep indent is harder to write and maintain. Using super deep indent encourages
longer lines and distupts the flow (you need to constantly switch from reading
top-down, to reading left-right). Small changes to code often cause multiline
diffs. It's harder to see the changes in the code. Also the motivation to do the
small change is lower, if in consequence you have to reformat the whole block.
You might thing that there is a better name for a variable, but you won't change
it because it would require reformatting next 10 lines.

While auto-formatting and diffs with hidden white-space changes helps, the problem
is still there. There is not much value in keeping the deep indent, as it doesn't
solve any real problem.

## Argument / method calls consistency

When passing multiple arguments or chaining many methods calls either put
all of those in a single line, or put each argument on a separate line. Do not mix
those two styles. If the method consists of many arguments or method calls,
strongly consider using multiline style even if the line is not too long. This
improves clarity, coveys complexity of the method and makes it easier to read.

Please keep in mind the rule about deep-indent, when splitting the arguments or
calls into multiple lines. Do not use deep indent.

### Method chaining

```ruby
# Good
User.active.fresh.limit(100)

# Good
User
  .active
  .fresh
  .limit(100)

# Bad
User.active
  .fresh.limit(100)
```

### Method arguments

```ruby
# Good
verify(user, order, shipper, product)

# Good
verify(
  user,
  order,
  shipper,
  product,
)

# Bad
verify(
  user, order,
  shipper, product,
)

# Bad
verify(user, order,
  shipper,
  product,
)
```

## Long lines

Avoid long lines if possible. Optimally, limit the length of line to 80 characters,
possibly lower value. See some rationale behind this in the generic Ruby Style guide
here: https://rubystyle.guide/#max-line-length

There are several issues with code packed into long lines:
- As mentioned in point related to deep-indent, longs break the flow of reading.
  You have to switch from reading top-down to reading left-right.

- Long lines hide the complexity of the code. Very compact code might suggest
  there is not much going on, but in reality, it might be very complex. There are
  reasons to keep the code broken into smaller parts. Forcing lots of logic into
  a single line games those rules, but doesn't solve the underlying problem.

- Code coverage tools might not be able to properly measure the coverage of the
  code. If there are multiple statements and branches in a single line, we will
  most likely get a false-positive coverage report.

- As with deep-indent, long lines are harder to write and maintain. Small changes
  to the code cause complex diffs. You can't as easily see which part of the code
  actually changed. You might be discouraged from making small changes.

## Visual separation

Do not glue large blocks of code together. Use empty lines to separate logical
parts. This makes it easier to read and follow the code. As a rule of thumb,
always use single empty line to separate code, which spans multiple lines. If a
method call spans multiple lines, separate it with a single empty line from the
previous and next statement. Do the same, if the block is passed to a method.

```ruby
# Bad
create(
  :order,
  description: "Some description",
  state: "shipped"
  date: Time.now,
)
user1 = create(:user, name: "John Doe")
user2 = create(:user, name: "John Doe")
create(
  :shipment,
  owner: user1,
  shipper: user2,
)

# Good
create(
  :order,
  description: "Some description",
  state: "shipped"
  date: Time.now,
)

user1 = create(:user, name: "John Doe")
user2 = create(:user, name: "John Doe")

create(
  :shipment,
  owner: user1,
  shipper: user2,
)
```
