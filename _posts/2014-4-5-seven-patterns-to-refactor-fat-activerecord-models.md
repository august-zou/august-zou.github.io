---
layout: post
title: Seven Patterns to Refactor Fat ActiveRecord Models(转载)
categories: [coding,Ruby]
tags: [ruby on rails]
fullview: true
---

[原文](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)

When teams use Code Climate to improve the quality of their Rails applications, they learn to break the habit of allowing models to get fat. “Fat models” cause maintenance issues in large apps. Only incrementally better than cluttering controllers with domain logic, they usually represent a failure to apply the Single Responsibility Principle (SRP). “Anything related to what a user does” is not a single responsibility.

Early on, SRP is easier to apply. ActiveRecord classes handle persistence, associations and not much else. But bit-by-bit, they grow. Objects that are inherently responsible for persistence become the de facto owner of all business logic as well. And a year or two later you have a User class with over 500 lines of code, and hundreds of methods in it’s public interface. Callback hell ensues.

As you add more intrinsic complexity (read: features!) to your application, the goal is to spread it across a coordinated set of small, encapsulated objects (and, at a higher level, modules) just as you might spread cake batter across the bottom of a pan. Fat models are like the big clumps you get when you first pour the batter in. Refactor to break them down and spread out the logic evenly. Repeat this process and you’ll end up with a set of simple objects with well defined interfaces working together in a veritable symphony.

You may be thinking:

`“But Rails makes it really hard to do OOP right!”`

I used to believe that too. But after some exploration and practice, I realized that Rails (the framework) doesn’t impede OOP all that much. It’s the Rails “conventions” that don’t scale, or, more specifically, the lack of conventions for managing complexity beyond what the Active Record pattern can elegantly handle. Fortunately, we can apply OO-based principles and best practices where Rails is lacking.


Don’t Extract Mixins from Fat Models

Let’s get this out of the way. I discourage pulling sets of methods out of a large ActiveRecord class into “concerns”, or modules that are then mixed in to only one model. I once heard someone say:

`“Any application with an app/concerns directory is concerning.”`

And I agree. Prefer composition to inheritance. Using mixins like this is akin to “cleaning” a messy room by dumping the clutter into six separate junk drawers and slamming them shut. Sure, it looks cleaner at the surface, but the junk drawers actually make it harder to identify and implement the decompositions and extractions necessary to clarify the domain model.

Now on to the refactorings!


## 1. Extract Value Objects

Value Objects are simple objects whose equality is dependent on their value rather than an identity. They are usually immutable. Date, URI, and Pathname are examples from Ruby’s standard library, but your application can (and almost certainly should) define domain-specific Value Objects as well. Extracting them from ActiveRecords is low hanging refactoring fruit.

In Rails, Value Objects are great when you have an attribute or small group of attributes that have logic associated with them. Anything more than basic text fields and counters are candidates for Value Object extraction.

For example, a text messaging application I worked on had a PhoneNumber Value Object. An e-commerce application needs a Money class. Code Climate has a Value Object named Rating that represents a simple A - F grade that each class or module receives. I could (and originally did) use an instance of a Ruby String, but Rating allows me to combine behavior with the data:

	class Rating
	  include Comparable
	
	  def self.from_cost(cost)
	    if cost <= 2
	      new("A")
	    elsif cost <= 4
	      new("B")
	    elsif cost <= 8
	      new("C")
	    elsif cost <= 16
	      new("D")
	    else
	      new("F")
	    end
	  end
	
	  def initialize(letter)
	    @letter = letter
	  end
	
	  def better_than?(other)
	    self > other
	  end
	
	  def <=>(other)
	    other.to_s <=> to_s
	  end
	
	  def hash
	    @letter.hash
	  end
	
	  def eql?(other)
	    to_s == other.to_s
	  end
	
	  def to_s
	    @letter.to_s
	  end
	end
Every ConstantSnapshot then exposes an instance of Rating in its public interface:

	class ConstantSnapshot < ActiveRecord::Base
	  # …
	
	  def rating
	    @rating ||= Rating.from_cost(cost)
	  end
	end
Beyond slimming down the ConstantSnapshot class, this has a number of advantages:

The #worse_than? and #better_than? methods provide a more expressive way to compare ratings than Ruby’s built-in operators (e.g. < and >).
Defining #hash and #eql? makes it possible to use a Rating as a hash key. Code Climate uses this to idiomatically group constants by their ratings using Enumberable#group_by.
The #to_s method allows me to interpolate a Rating into a string (or template) without any extra work.
The class definition provides a convenient place for a factory method, returning the correct Rating for a given “remediation cost” (the estimated time it would take to fix all of the smells in a given class).

## 2. Extract Service Objects

Some actions in a system warrant a Service Object to encapsulate their operation. I reach for Service Objects when an action meets one or more of these criteria:

The action is complex (e.g. closing the books at the end of an accounting period)
The action reaches across multiple models (e.g. an e-commerce purchase using Order, CreditCard and Customer objects)
The action interacts with an external service (e.g. posting to social networks)
The action is not a core concern of the underlying model (e.g. sweeping up outdated data after a certain time period).
There are multiple ways of performing the action (e.g. authenticating with an access token or password). This is the Gang of Four Strategy pattern.
As an example, we could pull a User#authenticate method out into a UserAuthenticator:

	class UserAuthenticator
	  def initialize(user)
	    @user = user
	  end
	
	  def authenticate(unencrypted_password)
	    return false unless @user
	
	    if BCrypt::Password.new(@user.password_digest) == unencrypted_password
	      @user
	    else
	      false
	    end
	  end
	end
And the SessionsController would look like this:


class SessionsController < ApplicationController
  def create
    user = User.where(email: params[:email]).first

    if UserAuthenticator.new(user).authenticate(params[:password])
      self.current_user = user
      redirect_to dashboard_path
    else
      flash[:alert] = "Login failed."
      render "new"
    end
  end
end

## 3. Extract Form Objects

When multiple ActiveRecord models might be updated by a single form submission, a Form Object can encapsulate the aggregation. This is far cleaner than using accepts_nested_attributes_for, which, in my humble opinion, should be deprecated. A common example would be a signup form that results in the creation of both a Company and a User:

	class Signup
	  include Virtus
	
	  extend ActiveModel::Naming
	  include ActiveModel::Conversion
	  include ActiveModel::Validations
	
	  attr_reader :user
	  attr_reader :company
	
	  attribute :name, String
	  attribute :company_name, String
	  attribute :email, String
	
	  validates :email, presence: true
	  # … more validations …
	
	  # Forms are never themselves persisted
	  def persisted?
	    false
	  end
	
	  def save
	    if valid?
	      persist!
	      true
	    else
	      false
	    end
	  end
	
	private
	
	  def persist!
	    @company = Company.create!(name: company_name)
	    @user = @company.users.create!(name: name, email: email)
	  end
	end
I’ve started using Virtus in these objects to get ActiveRecord-like attribute functionality. The Form Object quacks like an ActiveRecord, so the controller remains familiar:

class SignupsController < ApplicationController
  def create
    @signup = Signup.new(params[:signup])

    if @signup.save
      redirect_to dashboard_path
    else
      render "new"
    end
  end
end
This works well for simple cases like the above, but if the persistence logic in the form gets too complex you can combine this approach with a Service Object. As a bonus, since validation logic is often contextual, it can be defined in the place exactly where it matters instead of needing to guard validations in the ActiveRecord itself.


## 4. Extract Query Objects

For complex SQL queries littering the definition of your ActiveRecord subclass (either as scopes or class methods), consider extracting query objects. Each query object is responsible for returning a result set based on business rules. For example, a Query Object to find abandoned trials might look like this:

class AbandonedTrialQuery
  def initialize(relation = Account.scoped)
    @relation = relation
  end

  def find_each(&block)
    @relation.
      where(plan: nil, invites_count: 0).
      find_each(&block)
  end
end
You might use it in a background job to send emails:


AbandonedTrialQuery.new.find_each do |account|
  account.send_offer_for_support
end
Since ActiveRecord::Relation instances are first class citizens as of Rails 3, they make a great input to a Query Object. This allows you to combine queries using composition:


old_accounts = Account.where("created_at < ?", 1.month.ago)
old_abandoned_trials = AbandonedTrialQuery.new(old_accounts)
Don’t bother testing a class like this in isolation. Use tests that exercise the object and the database together to ensure the correct rows are returned in the right order and any joins or eager loads are performed (e.g. to avoid N + 1 query bugs).


## 5. Introduce View Objects

If logic is needed purely for display purposes, it does not belong in the model. Ask yourself, “If I was implementing an alternative interface to this application, like a voice-activated UI, would I need this?”. If not, consider putting it in a helper or (often better) a View object.

For example, the donut charts in Code Climate break down class ratings based on a snapshot of the codebase (e.g. Rails on Code Climate) and are encapsulated as a View:


	class DonutChart
	  def initialize(snapshot)
	    @snapshot = snapshot
	  end
	
	  def cache_key
	    @snapshot.id.to_s
	  end
	
	  def data
	    # pull data from @snapshot and turn it into a JSON structure
	  end
	end
I often find a one-to-one relationship between Views and ERB (or Haml/Slim) templates. This has led me to start investigating implementations of the Two Step View pattern that can be used with Rails, but I don’t have a clear solution yet.

Note: The term “Presenter” has caught on in the Ruby community, but I avoid it for its baggage and conflicting use. The “Presenter” term was introduced by Jay Fields to describe what I refer to above as “Form Objects”. Also, Rails unfortunately uses the term “view” to describe what are otherwise known as “templates”. To avoid ambiguity, I sometimes refer to these View objects as “View Models”.


## 6. Extract Policy Objects

Sometimes complex read operations might deserve their own objects. In these cases I reach for a Policy Object. This allows you to keep tangential logic, like which users are considered active for analytics purposes, out of your core domain objects. For example:

	class ActiveUserPolicy
	  def initialize(user)
	    @user = user
	  end
	
	  def active?
	    @user.email_confirmed? &&
	    @user.last_login_at > 14.days.ago
	  end
	end
This Policy Object encapsulates one business rule, that a user is considered active if they have a confirmed email address and have logged in within the last two weeks. You can also use Policy Objects for a group of business rules like an Authorizer that regulates which data a user can access.

Policy Objects are similar to Service Objects, but I use the term “Service Object” for write operations and “Policy Object” for reads. They are also similar to Query Objects, but Query Objects focus on executing SQL to return a result set, whereas Policy Objects operate on domain models already loaded into memory.


## 7. Extract Decorators

Decorators let you layer on functionality to existing operations, and therefore serve a similar purpose to callbacks. For cases where callback logic only needs to run in some circumstances or including it in the model would give the model too many responsibilities, a Decorator is useful.

Posting a comment on a blog post might trigger a post to someone’s Facebook wall, but that doesn’t mean the logic should be hard wired into the Comment class. One sign you’ve added too many responsibilities in callbacks is slow and brittle tests or an urge to stub out side effects in wholly unrelated test cases.

Here’s how you might extract Facebook posting logic into a Decorator:

	class FacebookCommentNotifier
	  def initialize(comment)
	    @comment = comment
	  end
	
	  def save
	    @comment.save && post_to_wall
	  end
	
	private
	
	  def post_to_wall
	    Facebook.post(title: @comment.title, user: @comment.author)
	  end
	end
	And how a controller might use it:
	
	class CommentsController < ApplicationController
	  def create
	    @comment = FacebookCommentNotifier.new(Comment.new(params[:comment]))
	
	    if @comment.save
	      redirect_to blog_path, notice: "Your comment was posted."
	    else
	      render "new"
	    end
	  end
	end
Decorators differ from Service Objects because they layer on responsibilities to existing interfaces. Once decorated, collaborators just treat the FacebookCommentNotifier instance as if it were a Comment. In its standard library, Ruby provides a number of facilities to make building decorators easier with metaprogramming.

Wrapping Up

Even in a Rails application, there are many tools to manage complexity in the model layer. None of them require you to throw out Rails. ActiveRecord is a fantastic library, but any pattern breaks down if you depend on it exclusively. Try limiting your ActiveRecords to persistence behavior. Sprinkle in some of these techniques to spread out the logic in your domain model and the result will be a much more maintainable application.

You’ll also note that many of the patterns described here are quite simple. The objects are just “Plain Old Ruby Objects” (PORO) used in different ways. And that’s part of the point and the beauty of OOP. Every problem doesn’t need to be solved by a framework or library, and naming matters a great deal.

What do you think of the seven techniques I presented above? What are your favorites and why? Have I missed any? Let me know in the comments!

P.S. If you found this post useful, you may want to subscribe to our email newsletter (see below). It’s low volume, and includes content about OOP and refactoring Rails applications like this.
