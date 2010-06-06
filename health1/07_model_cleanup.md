
!SLIDE

    Once we are convinced that we have a solid
    characterization of the models and the 
    acts_as_ass behavior, we can aggressively
    refactor.

!SLIDE code smallest

    Author: rick

    Beginning the process of merging the EOB/Payment/Charge 
    classes into the Entry class hierarchy ...
    
    Created the Entry spec, then the Entry class.  Redirected 
    the parent of Charge to be Entry.  Then created the entries 
    table.  Then brought (copy, not move) the column definitions 
    from the charges table into the entries table.  This has not 
    moved the Charge data into the Entry table yet -- that will 
    be at a later phase.
    
    So, some nasty gank-code in acts_as_associatable was caught 
    by our characterization tests.  Its author for some reason 
    decided to hard-wire the table names in the queries in there 
    for possible associations, so they have to be changed manually, 
    as we move each class out.  I can't say enough bad things about 
    this plugin.  I'm just glad we're now actively under-way to 
    getting rid of that code.
    
    Merry Xmas, yo.

!SLIDE code smallest

    @@@diff
    --- a/app/models/charge.rb
    +++ b/app/models/charge.rb
    @@ -22,7 +22,7 @@
     #  account_id           :integer(11)   
     #
     
    -class Charge < ActiveRecord::Base
    +class Charge < Entry
     
    --- /dev/null
    +++ b/app/models/entry.rb
    @@ -0,0 +1,2 @@
    +class Entry < ActiveRecord::Base
    +end
    
    --- /dev/null
    +++ b/db/migrate/505_create_entries_table.rb
    @@ -0,0 +1,11 @@
    +class CreateEntriesTable < ActiveRecord::Migration
    +end
    diff --git a/db/migrate/506_merge_charge_schema_to_entries.rb b/db/migrate/506_merge_charge_schema_to_entries.rb
    new file mode 100644
    index 0000000..7285cc7
    --- /dev/null
    +++ b/db/migrate/506_merge_charge_schema_to_entries.rb
    @@ -0,0 +1,45 @@
    +class MergeChargeSchemaToEntries < ActiveRecord::Migration
    +  def self.up
    +    # add columns
    +    add_column :entries, "type",                 :string
       # ...
    
    --- a/spec/models/charge_spec.rb
    +++ b/spec/models/charge_spec.rb
    @@ -1,6 +1,16 @@
    +describe Charge do
    +  it "should be a kind of entry" do

!SLIDE code smallest

    @@@diff
    --- a/vendor/plugins/acts_as_associatable/lib/acts_as_associatable.rb
    +++ b/vendor/plugins/acts_as_associatable/lib/acts_as_associatable.rb
    @@ -214,20 +214,21 @@ module ActsAsAssociatable
                 # NOTE:  this is only called by the "step two" controller nonsense.
                 possibles = []
                 possibles += Charge.find :all,
    -                 :select => 'charges.*',
    -                 :joins => 'JOIN accepted_health_terms ON (charges.health_issue_term_id = accepted_health_terms.id ) JOIN health_terms ON (accepted_health_terms.health_term_id = health_terms.id )',
    +                 :select => 'entries.*',
    +                 :joins => 'JOIN accepted_health_terms ON (entries.health_issue_term_id = accepted_health_terms.id ) JOIN health_terms ON (accepted_health_terms.health_term_id = health_terms.id )',
                      :conditions => [
    -                  ( source.id ? 'charges.id != ? AND ' : '? IS NULL AND ' ) +
    -                  'charges.account_id = ? AND ( ' +
    -                  '( charges.person_id = ? AND health_terms.text = ? AND ABS( DATEDIFF( charges.service_date, ? ) ) < 7 ) OR '+
    -                  '( charges.person_id = ? AND charges.provider_id = ? AND ABS( DATEDIFF( charges.service_date, ? ) ) < 7 ) OR '+
    -                  '( charges.person_id = ? AND charges.provider_id = ? AND health_terms.text = ? ) ) ',
    +                  ( source.id ? 'entries.id != ? AND ' : '? IS NULL AND ' ) +
    +                  "entries.type = 'Charge' AND " + 
    +                  'entries.account_id = ? AND ( ' +
    +                  '( entries.person_id = ? AND health_terms.text = ? AND ABS( DATEDIFF( entries.service_date, ? ) ) < 7 ) OR '+
    +                  '( entries.person_id = ? AND entries.provider_id = ? AND ABS( DATEDIFF( entries.service_date, ? ) ) < 7 ) OR '+
    +                  '( entries.person_id = ? AND entries.provider_id = ? AND health_terms.text = ? ) ) ',
                       source.id,
                       source.account_id,
                       source.person_id, source.health_term, source.service_date,
                       source.person_id, source.provider_id, source.service_date,
                       source.person_id, source.provider_id, source.health_term ],
    -                :order => 'charges.record_id'
    +                :order => 'entries.record_id'
                 possibles += Payment.find :all,
                      :select => 'payments.*',
