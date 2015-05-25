  # Mailboxer
  
  
  I needed private messaging, found the Mailboxer (https://github.com/mailboxer/mailboxer) very good, but in my system I use Mongoid, this gem was not going to serve me (she makes heavy use of joins and MongoDB does not support this, nor is there need), I realized that to wear the jewelry that I needed to encode, so I decided to put their hands dirty, I mean, in the code and porting it to support Mongoid, so this is a gem Mailboxer supporting Mongoid and only this.
  I intend to keep that gem always compatible with Mailboxer and can add new features.
  I thank the team behind the Mailboxer and those who contributed to make it big (I love it).
  
  Sorry for the Google English
  
  Installation
  ------------
  
  Add to your Gemfile:
  
  ```ruby
  gem 'mailboxer-mongoid',git: 'https://github.com/Malohka/mailboxer-mongoid'
  ```
  
  Then run:
  
  ```sh
  $ bundle install
  ```
  
  Run install script:
  
  ```sh
  $ rails g mailboxer:install
  ```
  
  no need for migration (great)
  ```
  
  You can also generate email views:
  
  ```sh
  $ rails g mailboxer:views
  ```
  
  ## Requirements & Settings
  
  ### Emails
  
  We are now adding support for sending emails when a Notification or a Message is sent to one or more recipients. You should modify the mailboxer initializer (/config/initializer/mailboxer.rb) to edit these settings.
  
  ```ruby
  Mailboxer.setup do |config|
    #Enables or disables email sending for Notifications and Messages
    config.uses_emails = true
    #Configures the default `from` address for the email sent for Messages and Notifications of Mailboxer
    config.default_from = "no-reply@dit.upm.es"
    ...
  end
  ```
  
  You can change the way in which emails are delivered by specifying a custom implementation of notification and message mailers
  
  ```ruby
  Mailboxer.setup do |config|
    config.notification_mailer = CustomNotificationMailer
    config.message_mailer = CustomMessageMailer
    ...
  end
  ```
  
  ### User identities
  
  Users must have an identity defined by a `name` and an `email`. We must ensure that Messageable models have some specific methods. These methods are:
  
  ```ruby
  #Returning any kind of identification you want for the model
  def name
    return "You should add method :name in your Messageable model"
  end
  ```
  
  ```ruby
  #Returning the email address of the model if an email should be sent for this object (Message or Notification).
  #If no mail has to be sent, return nil.
  def mailboxer_email(object)
    #Check if an email should be sent for that object
    #if true
    return "define_email@on_your.model"
    #if false
    #return nil
  end
  ```
  
  These names are explicit enough to avoid colliding with other methods, but as long as you need to change them you can do it by using mailboxer initializer (/config/initializer/mailboxer.rb). Just add or uncomment the following lines:
  
  ```ruby
  Mailboxer.setup do |config|
    # ...
    #Configures the methods needed by mailboxer
    config.email_method = :mailboxer_email
    config.name_method = :name
    # ...
  end
  ```
  
  You may change whatever you want or need. For example:
  
  ```ruby
  config.email_method = :notifications_email
  config.name_method = :display_name
  ```
  
  Will use the method `notification_email(object)` instead of `mailboxer_email(object)` and `display_name` for `name`.
  
  Using default or custom method names, if your model doesn't implement them, Mailboxer will use dummy methods so as to notify you of missing methods rather than crashing.
  
  ## Preparing your models
  
  In your model:
  
  ```ruby
  class User < ActiveRecord::Base
    acts_as_messageable
  end
  ```
  
  You are not limited to the User model. You can use Mailboxer in any other model and use it in serveral different models. If you have ducks and cylons in your application and you want to exchange messages as if they were the same, just add `acts_as_messageable` to each one and you will be able to send duck-duck, duck-cylon, cylon-duck and cylon-cylon messages. Of course, you can extend it for as many classes as you need.
  
  Example:
  
  ```ruby
  class Duck < ActiveRecord::Base
    acts_as_messageable
  end
  ```
  
  ```ruby
  class Cylon < ActiveRecord::Base
    acts_as_messageable
  end
  ```
  
  ## Mailboxer API
  
  ### How can I send a message?
  
  ```ruby
  #alfa wants to send a message to beta
  alfa.send_message(beta, "Body", "subject")
  ```
  
  ### How can I reply a message?
  
  ```ruby
  #alfa wants to reply to all in a conversation
  #using a receipt
  alfa.reply_to_all(receipt, "Reply body")
  
  #using a conversation
  alfa.reply_to_conversation(conversation, "Reply body")
  ```
  
  ```ruby
  #alfa wants to reply to the sender of a message (and ONLY the sender)
  #using a receipt
  alfa.reply_to_sender(receipt, "Reply body")
  ```
  
  ### How can I retrieve my conversations?
  
  ```ruby
  #alfa wants to retrieve all his conversations
  alfa.mailbox.conversations
  
  #A wants to retrieve his inbox
  alfa.mailbox.inbox
  
  #A wants to retrieve his sent conversations
  alfa.mailbox.sentbox
  
  #alfa wants to retrieve his trashed conversations
  alfa.mailbox.trash
  ```
  
  ### How can I delete a message from trash?
  
  ```ruby
  #delete conversations forever for one receipt (still in database)
  receipt.mark_as_deleted
  
  #you can mark conversation as deleted for one participant
  conversation.mark_as_deleted participant
  
  #Mark the object as deleted for messageable
  #Object can be:
    #* A Receipt
    #* A Conversation
    #* A Notification
    #* A Message
    #* An array with any of them
  alfa.mark_as_deleted conversation
  ```
  
  ### How can I paginate conversations?
  
  You can use Kaminari to paginate the conversations as normal. Please, make sure you use the last version as mailboxer uses `select('DISTINCT conversations.*')` which was not respected before Kaminari 0.12.4 according to its changelog. Working corretly on Kaminari 0.13.0.
  
  ```ruby
  #Paginating all conversations using :page parameter and 9 per page
  conversations = alfa.mailbox.conversations.page(params[:page]).per(9)
  
  #Paginating received conversations using :page parameter and 9 per page
  conversations = alfa.mailbox.inbox.page(params[:page]).per(9)
  
  #Paginating sent conversations using :page parameter and 9 per page
  conversations = alfa.mailbox.sentbox.page(params[:page]).per(9)
  
  #Paginating trashed conversations using :page parameter and 9 per page
  conversations = alfa.mailbox.trash.page(params[:page]).per(9)
  ```
  
  ### How can I read the messages of a conversation?
  
  As a messageable, what you receive are receipts, which are associated with the message itself. You should retrieve your receipts for the conversation a get the message associated with them.
  
  This is done this way because receipts save the information about the relation between messageable and the messages: is it read?, is it trashed?, etc.
  
  ```ruby
  #alfa gets the last conversation (chronologically, the first in the inbox)
  conversation = alfa.mailbox.inbox.first
  
  #alfa gets it receipts chronologically ordered.
  receipts = conversation.receipts_for alfa
  
  #using the receipts (i.e. in the view)
  receipts.each do |receipt|
    ...
    message = receipt.message
    read = receipt.is_unread? #or message.is_unread?(alfa)
    ...
  end
  ```
