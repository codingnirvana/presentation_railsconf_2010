
!SLIDE

    Core data model


!SLIDE larger

[ the primary purpose of the application ]

!SLIDE larger

[ tracking medical payments, bills, eob's, etc. ... ]

!SLIDE larger

[ mis-modelled. ]

!SLIDE code smallest

    @@@ruby
    
    class Payment < ActiveRecord::Base
      acts_as_taggable
      acts_as_associatable
      attr_human_name 'person_id' => 'Family Member'
      attr_human_name 'health_issue' => 'Reason for Visit'
      belongs_to :pre_tax_account
      belongs_to :deductible_expense
      belongs_to :user
      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_payments
      has_many    :notes, :as => :association, :order => 'filed_on'
      belongs_to  :health_issue_accepted_term, :foreign_key => 'health_issue_term_id', :class_name => 'AcceptedHealthTerm'
      before_save   :set_defaults
      validates_presence_of :paid_on, :if => :step_three?
      validates_date :service_date, :if => :service_date?
      validates_date_time :created_on, :if => :created_on?
      validates_date :paid_on, :if => :paid_on?
    
      def self.type
        'payment'
      end
    
      def type
        self.class.type
      end
    
      def occurred_on
        paid_on
      end
    
      def self.controller_name
        'payments'
      end

      # ...

!SLIDE code smallest

    @@@ruby

    class Payment < ActiveRecord::Base
      
      acts_as_associatable
      
      
      
      
      
      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_payments
      has_many    :notes, :as => :association, :order => 'filed_on'
      
      
      validates_presence_of :paid_on, :if => :step_three?
      validates_date :service_date, :if => :service_date?
      validates_date_time :created_on, :if => :created_on?
      validates_date :paid_on, :if => :paid_on?

      def self.type
        'payment'
      end

      
      
      

      def occurred_on
        paid_on
      end

      def self.controller_name
        'payments'
      end

      # ...    
    
!SLIDE code smallest

    @@@ruby
    class Charge < ActiveRecord::Base
      acts_as_taggable
      acts_as_associatable
      attr_human_name 'person_id' => 'Family Member'
      attr_human_name 'health_issue' => 'Reason for Visit'
      attr_human_name 'billed_on' => 'Billing Date'
      FILED_BY_OPTIONS = ['Provider', 'Me', 'Not applicable']
      belongs_to :actual_servicetype, :class_name=>"Servicetype", :foreign_key=>"servicetype_id"
      belongs_to :user
      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_charges
      has_many    :notes, :as => :association, :order => 'filed_on'
      validates_presence_of :billed_on, :if => :step_three?
      validates_presence_of :amount_in_cents, :if => :step_three?
      validates_date :service_date, :if => :service_date?
      validates_date :billed_on, :if => :billed_on?
      validates_presence_of :servicetype, :if => :step_three?
      validates_associated :actual_servicetype, :if => :step_three?
    
      def self.type
        'bill'
      end
    
      def type
        self.class.type
      end
    
      def occurred_on
        billed_on
      end
    
      def self.controller_name
        'bills'
      end
    
      # ...

!SLIDE code smallest

    @@@ruby
    class Charge < ActiveRecord::Base
      
      acts_as_associatable
      
      
      
      
      
      
      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_charges
      has_many    :notes, :as => :association, :order => 'filed_on'
      validates_presence_of :billed_on, :if => :step_three?
      validates_presence_of :amount_in_cents, :if => :step_three?
      
      
      validates_presence_of :servicetype, :if => :step_three?
      validates_associated :actual_servicetype, :if => :step_three?

      def self.type
        'bill'
      end

      
      
      

      def occurred_on
        billed_on
      end

      def self.controller_name
        'bills'
      end

      # ...    
    
!SLIDE code smallest

    @@@ruby
    class ExplanationOfBenefit < ActiveRecord::Base
      acts_as_taggable
      acts_as_associatable
      attr_human_name 'person_id' => 'Family Member'
      attr_human_name 'health_issue' => 'Reason for Visit'
      belongs_to :insurance  
      belongs_to :user
      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_eobs
      has_many    :notes, :as => :association, :order => 'filed_on'
      validates_presence_of :date, :if => :step_three?
      validates_date :service_date, :if => :service_date?
      validates_date :date, :if => :date?
      validates_date :paid_on, :if => :paid_on?
         
      def self.type
        'EOB'
      end
    
      def type
        self.class.type
      end
    
      def occurred_on
        date
      end
    
      def self.controller_name
        'explanation_of_benefits'
      end

      # ...    
    
!SLIDE code smallest

    @@@ruby
    class ExplanationOfBenefit < ActiveRecord::Base
      
      acts_as_associatable
      
      
      
      
      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_eobs
      has_many    :notes, :as => :association, :order => 'filed_on'
      validates_presence_of :date, :if => :step_three?
      validates_date :service_date, :if => :service_date?
      validates_date :date, :if => :date?
      validates_date :paid_on, :if => :paid_on?

      def self.type
        'EOB'
      end

      
      
      

      def occurred_on
        date
      end

      def self.controller_name
        'explanation_of_benefits'
      end

      # ...    
    

!SLIDE

    Right next to each other...


!SLIDE code smallest

    @@@ruby
    class Charge < ActiveRecord::Base
      
      acts_as_associatable
      
      
      
      
      
      
      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_charges
      has_many    :notes, :as => :association, :order => 'filed_on'
      validates_presence_of :billed_on, :if => :step_three?
      validates_presence_of :amount_in_cents, :if => :step_three?
      
      
      validates_presence_of :servicetype, :if => :step_three?
      validates_associated :actual_servicetype, :if => :step_three?

      def self.type
        'bill'
      end

      
      
      

      def occurred_on
        billed_on
      end

      def self.controller_name
        'bills'
      end



!SLIDE code smallest

    @@@ruby

    class Payment < ActiveRecord::Base

      acts_as_associatable





      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_payments
      has_many    :notes, :as => :association, :order => 'filed_on'


      validates_presence_of :paid_on, :if => :step_three?
      validates_date :service_date, :if => :service_date?
      validates_date_time :created_on, :if => :created_on?
      validates_date :paid_on, :if => :paid_on?

      def self.type
        'payment'
      end





      def occurred_on
        paid_on
      end

      def self.controller_name
        'payments'
      end


!SLIDE code smallest

    @@@ruby
    class ExplanationOfBenefit < ActiveRecord::Base

      acts_as_associatable




      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_eobs
      has_many    :notes, :as => :association, :order => 'filed_on'
      validates_presence_of :date, :if => :step_three?
      validates_date :service_date, :if => :service_date?
      validates_date :date, :if => :date?
      validates_date :paid_on, :if => :paid_on?

      def self.type
        'EOB'
      end





      def occurred_on
        date
      end

      def self.controller_name
        'explanation_of_benefits'
      end

!SLIDE smaller

    Even blindly looking at code these models are 
    clearly related.


    Conceptually they're tightly related as well.


    They should be in the same class hierarchy.


    But changing the core of the application is 
    hard to justify.

!SLIDE code smallest

    @@@ruby

    # == Schema Information
    # Schema version: 473
    #
    # Table name: associations_payments
    #
    #  id               :integer(11)   not null, primary key
    #  associatable_id  :integer(11)   
    #  association_id   :integer(11)   
    #  association_type :string(255)   
    #
    
    class AssociationsPayment < ActiveRecord::Base
      belongs_to :associatable, :class_name => 'Payment', :foreign_key => 'id'
      belongs_to :associations, :polymorphic => true
    end

    class Payment < ActiveRecord::Base
      # ...
      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_payments
      # ...
    end

!SLIDE code smallest

    @@@ruby

    # == Schema Information
    # Schema version: 473
    #
    # Table name: associations_charges
    #
    #  id               :integer(11)   not null, primary key
    #  associatable_id  :integer(11)   
    #  association_id   :integer(11)   
    #  association_type :string(255)   
    #
    
    class AssociationsCharge < ActiveRecord::Base
      belongs_to :associatable, :class_name => 'Charge', :foreign_key => 'id'
      belongs_to :associations, :polymorphic => true
    end

    class Charge < ActiveRecord::Base
      # ...
      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_charges
      # ...
    end

!SLIDE code smallest

    @@@ ruby 
    
    # == Schema Information
    # Schema version: 473
    #
    # Table name: associations_eobs
    #
    #  id               :integer(11)   not null, primary key
    #  associatable_id  :integer(11)   
    #  association_id   :integer(11)   
    #  association_type :string(255)   
    #
    
    class AssociationsEob < ActiveRecord::Base
      belongs_to :associatable, :class_name => 'ExplanationOfBenefit', :foreign_key => 'id'
      belongs_to :associations, :polymorphic => true
    end

    class ExplanationOfBenefit < ActiveRecord::Base
      # ...
      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_eobs
      # ...
    end

!SLIDE larger

[ TODO: domain model image ]
[ TODO: domain model image with "associations" ]


!SLIDE

    even the linkages between them
    suffer from failure to use OO
    to help model related concepts
