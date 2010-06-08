
!SLIDE

    So what did things look like
    after a 300-commit month?

!SLIDE code smallest

    @@@ruby
    class Entry < ActiveRecord::Base
      attr_accessor :step
      belongs_to :user
      belongs_to :record
      has_many   :notes, :as => :association, :order => 'filed_on'
    
      def associations
        return [] unless record
        record.entries - [ self ]
      end
      
      def associate(*other_entries)
        raise "Cannot associate unless Entry has a Record" unless record
        entries_without_records, entries_with_records = other_entries.partition {|entry| entry.record.nil? }
        entries_without_records.each {|entry| entry.update_attributes(:record => record) }
        unless entries_with_records.empty?
          other_records = entries_with_records.collect(&:record)
          record.take_entries_from(*other_records)
        end
      end
      
      def disassociate(*other_entries)
        other_entries.each { |entry|  entry.update_attributes(:record => nil) }
      end
      
      def mileage # ...
      def mileage=( new_mileage ) # ...
      def mileage_rate # ...
    
      # ... other small business methods carried over from a_a_a
    
!SLIDE code smallest

    @@@ruby
      before_save       :ensure_record_association
      after_destroy     :notify_record_of_removal
      after_update      :notify_changed_record_of_removal  

      def ensure_record_association
        if record.blank?
          self.record = Record.create
        end
      end

      def notify_record_of_removal
        record.entry_removed
      end
      
      def notify_changed_record_of_removal
        if old_record and old_record != record
          old_record.entry_removed
        end
      end

!SLIDE code smallest

    @@@ruby
    class Record < ActiveRecord::Base
      has_many :entries  # see 'associations' method below for what :order to add once all entry types could have the same date attribute
      has_many :charges, :order => 'billed_on'
      has_many :eobs, :order => 'date'
      has_many :payments, :order => 'paid_on'
      before_save :invalidate_eob_based_provider_medstimates
    
      def entry_removed
        if entries.blank?
          destroy
        end
      end
      
      def take_entries_from(*others)
        raise ArgumentError if others.blank?
        raise TypeError, "expected Records, but got #{others.collect(&:class).inspect}" if others.any? { |arg|  !arg.is_a?(self.class) }
        
        self.class.transaction do
          others.uniq.each do |rec|
            rec.entries.each do |ent|
              ent.update_attributes(:record => self)
            end
          end
        end
      end
        
      def associations
        entries.sort do |a,b|
          if a.occurred_on and b.occurred_on
            a.occurred_on <=> b.occurred_on
          else
            a.id <=> b.id
          end
        end
      end
      
      # ... other business-related methods

!SLIDE code smallest

    @@@ruby
    class Charge < Entry
      attr_human_name 'billed_on' => 'Billing Date'
      FILED_BY_OPTIONS = ['Provider', 'Me', 'Not applicable']
      belongs_to :actual_servicetype, :class_name=>"Servicetype", :foreign_key=>"servicetype_id"
      validates_presence_of :billed_on, :if => :step_three?
      validates_presence_of :amount_in_cents, :if => :step_three?
      validates_date :service_date, :if => :service_date?
      validates_date :billed_on, :if => :billed_on?
      validates_presence_of :servicetype, :if => :step_three?
      validates_associated :actual_servicetype, :if => :step_three?
          
      def occurred_on
        billed_on
      end
    
      def controller_name
        self.class.controller_name
      end
      
      def note_summary  # ...
      def event_date  # ...
      def servicetype_searchable   # ...
      def servicetype_picked  # ...    
      def servicetype_picked=( picked )   # ...
    end  

!SLIDE code smallest
    @@@ruby
    class EntriesController < ApplicationController
      # GET /entries
      # GET /entries.xml
      def index  # ...
    
      # GET /entries/1
      # GET /entries/1.xml
      def show  # ...
    
      # GET /entries/new
      def new  # ...
    
      # GET /entries/1;edit
      def edit  # ...
    
      # POST /entries
      # POST /entries.xml
      def create  # ...
    
      # PUT /entries/1
      # PUT /entries/1.xml
      def update  # ...
    
      # DELETE /entries/1
      # DELETE /entries/1.xml
      def destroy  # ...

      def search  # ...
    
      def inline_new_person  # ... 
      def inline_people_form  # ...  
      def inline_new_provider  # ...
      # ... more 'render a partial' methods
    end

!SLIDE code smallest
    @@@ruby
    class EobsController < EntriesController
      helper :insurances
    end



    
    class PaymentsController < EntriesController
      before_filter :fetch_deductible_expenses, :only => [ :new, :edit ]
    
      def fetch_deductible_expenses
        result = DeductibleExpense.find(:all, :order => :name)
        default, rest  = result.partition {|item| item.name == 'Generic Deductible'}
        @deductible_expenses = default + rest
      end
      private :fetch_deductible_expenses
    
      def auto_complete_for_payment_paid_to
        paid_to = (params[:entry] || {})[:paid_to]
        @providers = current_user.providers.find_name_like(paid_to)
        render :layout => false
      end
    end

!SLIDE code smallest

    @@@ruby
		class BillsController < EntriesController 
		  include Ziya
		  # included for the @license class variable
		  include ZiyaHelpers

		  before_filter :account_required
		  before_filter :setup_conditions_for_balance, :only => [:summary, :byfamilymember, :byprovider, :byreasonforvisit, :byservicedate]

		  def build_reimbursement_bar_chart # ... 
		  def summary # ... 
		  def byfamilymember  # ... 
		  def byprovider  # ... 
		  def byreasonforvisit  # ... 
		  def byservicedate  # ... 
		  def transaction_log  # ... 
		  def show_for_service_date  # ... 

		private

		  def setup_conditions_for_balance  # ...  
		  def calculate_average_account_reimbursement  # ... 
		  def calculate_average_site_reimbursement  # ... 
		  def calculate_average_reimbursement  # ... 
		end
     