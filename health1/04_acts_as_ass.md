
!SLIDE

    Acts as Ass

!SLIDE code smallest

    @@@ruby

    class Payment < ActiveRecord::Base
      # ...
      acts_as_associatable
      # ...
      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_payments
      # ...
      has_many    :notes, :as => :association, :order => 'filed_on'
      # ...
    end

!SLIDE code smallest

    @@@ruby

    class Charge < ActiveRecord::Base
      # ...
      acts_as_associatable
      # ...
      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_charges
      # ...
      has_many    :notes, :as => :association, :order => 'filed_on'
      # ...
    end

!SLIDE code smallest

    @@@ruby

    class ExplanationOfBenefit < ActiveRecord::Base
      # ...
      acts_as_associatable
      # ...
      has_many_polymorphs :associations, :from => [ :payments, :charges, 
         :explanation_of_benefits ], :as => :associatable, 
         :through => :associations_eobs
	    # ...
      has_many    :notes, :as => :association, :order => 'filed_on'
      # ...
    end

!SLIDE

    looking for acts_as_ass elsewhere
    it turned up in CONTROLLERS(!)

!SLIDE code smallest

    @@@ruby
    
    class PaymentsController < ApplicationController
      # ...
    
      acts_as_associatable :model => Payment, :param => :payment
    
      # ...
    
      def create
        @payment = Payment.new( params[:payment] )
    
        # ...
    
        @payment.step = 3
        begin
          @payment.save!
          if params[:association]
            params[:association].each do |p|
              if p[:associate] == 'on'
                case p[:type]
                  when 'bill'
                    assoc_class = Charge
                  when 'payment'
                    assoc_class = Payment
                  when 'EOB'
                    assoc_class = ExplanationOfBenefit
                end
                @payment.associate( assoc_class.find( p[:id], :conditions => [ 'account_id = ?', current_user.account_id ] ) )
              end
            end
          end
    
        # ... it only gets worse
			
!SLIDE code smallest

    @@@ruby

    class ExplanationOfBenefitsController < ApplicationController
      # ...

      acts_as_associatable :model => ExplanationOfBenefit, :param => :explanation_of_benefit

      # ...    
    
      def create
        @eob = ExplanationOfBenefit.new( params[:explanation_of_benefit] )

        # ...    
    
        @eob.step = 3
        begin
          @eob.save!
          if params[:association]
            params[:association].each do |p|
              if p[:associate] == 'on'
                case p[:type]
                  when 'bill'
                    assoc_class = Charge
                  when 'payment'
                    assoc_class = Payment
                  when 'EOB'
                    assoc_class = ExplanationOfBenefit
                end
                @eob.associate(assoc_class.find(p[:id], :conditions => [ 'user_id = ?', current_user.id ]))
              end
            end
          end

        # ... it only gets worse

!SLIDE code smallest

    @@@ruby

    class BillsController < ApplicationController
    	# ...

      acts_as_associatable :model => Charge, :param => :charge

      # ...
    
      def create
        @charge = Charge.new(params[:charge])

        # ... 

        @charge.step = 3
        begin
          @charge.save!
          if params[:association]
            params[:association].each do |p|
              if p[:associate] == 'on'
                case p[:type]
                  when 'bill'
                    assoc_class = Charge
                  when 'payment'
                    assoc_class = Payment
                  when 'EOB'
                    assoc_class = ExplanationOfBenefit
                end
                @charge.associate(assoc_class.find(p[:id], :conditions => [ 'account_id = ?', current_user.account.id ]))
              end
            end
          end

        # ... it only gets worse

!SLIDE

    Also, it a "Bill" or a "Charge"?

!SLIDE code smallest

    @@@ruby
    
    class BillsController < ApplicationController
      #   ^^^^^
    
      acts_as_associatable :model => Charge, :param => :charge
    
      #                              ^^^^^^
    
    
      def create
        @charge = Charge.new(params[:charge])
    
        #         ^^^^^^


!SLIDE

               If you aren't cringing, 
               please go now and 
               shower Eric Evans
               with your $$$ for a copy 
               of Domain Driven Design.

<br/>
<br/>
<br/>

<div class="ddd">
	<a href="http://domaindrivendesign.org/books#DDD"><img src="ddd_cover.jpg"></a>
</div>

!SLIDE full-page

<img src="acts_as_ass_textmate.png">



    
    