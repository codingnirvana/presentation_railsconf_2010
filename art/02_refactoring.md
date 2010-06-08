
!SLIDE

    In a controller needing 30-60+
    seconds to render, there was a 
    frequently used setup method.

!SLIDE

    The real irony:

!SLIDE  code smallest

    # The following lines take about 0.01 seconds a piece.  
    # They don't seems to recover that time in speed.  I'm
    # leaving them in just in case someone else wants 
    # to experiment later.

!SLIDE code smallest
    @@@ruby
    def available_products_init
      if params[:category]
        category_string = params[:category].to_s.strip.downcase
        params[:category] = case
                            when category_string == 'xxx' then '1'
                            when category_string == 'xxx' then '2'
        # ...
      end
      
      popular_sales = !params[:popularity].blank? && ('1'..'2').include?( params[:popularity] )
      popular_xxxxxxs = !params[:popularity].blank? && ('3'..'4').include?( params[:popularity] )
      if popular_sales
        xxxxxx_sql = %q{
            SELECT XXXXX.xxxxxx_id, SUM( xxxxxx.quantity ) AS xxxxxx
            FROM xxxxxx
              INNER JOIN XXXXX ON xxxxxx.product_id = XXXXX.id
        }
        case params[:popularity]
          when '1' then xxxxxx_sql += 'AND xxxxxx.created_on >= current_date - 7 '
          when '2' then xxxxxx_sql += 'AND xxxxxx.created_on >= current_date - 30 '
        end
        xxxxxx_sql += %q{
            GROUP BY XXXXX.xxxxxx_id
        }
      end
      xxxxxx_xxxxxx_sql = %q{
          SELECT xxxxxx_xxxxxx.xxxxxx_id,
            MAX( CASE xxxxxx_xxxxxx.xxxxxxning_date <= current_date WHEN true THEN xxxxxx_xxxxxx.xxxxxxning_date ELSE NULL END ) AS newest_xxxxxx
          FROM xxxxxx_xxxxxx
      }
      if popular_xxxxxxs
        case params[:popularity]
          when '3' then xxxxxx_xxxxxx_sql += "WHERE ( xxxxxx_xxxxxx.xxxxxx_current_xxxxxxner_type = 'week' ) "
          when '4' then xxxxxx_xxxxxx_sql += "WHERE ( xxxxxx_xxxxxx.xxxxxx_current_xxxxxxner_type = 'month' ) "
        end
      end
      # ...
      XXXXX.connection.execute( 'DROP TABLE IF EXISTS available_XXXXX' )
      XXXXX.connection.execute( available_XXXXX_create )


!SLIDE

    This setup method was CREATING
    A TEMPORARY DATABASE TABLE ON
    EVERY CALL.

    This was one of the most-accessed
    methods in the entire application.

!SLIDE code smallest    
    @@@diff
    Author: rick
    
        Extracting a mockable inspection point for the insane temp-table creation SQL.
    
    --- a/app/controllers/shop_controller.rb
    +++ b/app/controllers/shop_controller.rb
    @@ -68,6 +68,12 @@ class ShopController < ApplicationController
         return str.to_i if str.to_i > 0
         nil
       end
    +
    +  # execute a temporary table creation SQL string, dropping the available_products table if it already exists
    +  def build_temporary_available_products_table(creation_sql)
    +    Product.connection.execute('DROP TABLE IF EXISTS available_products')
    +    Product.connection.execute(creation_sql)
    +  end
    
       def available_products_init
         # convert category string into numeric category
    @@ -185,8 +191,7 @@ class ShopController < ApplicationController
           end
    
         # actually run the built SQL to create the temporary table
    -        Product.connection.execute( 'DROP TABLE IF EXISTS available_products' )
    -        Product.connection.execute( available_products_create )
    +    build_temporary_available_products_table(available_products_create)
    
         # now build SQL to retrieve the designs of interest from the temporary table
             available_designs_sql = %q{
    
!SLIDE code smallest

    @@@diff	
    Author: rick
    
        Literal characterization of temp-table creation code.
        
    --- a/spec/controllers/shop_controller_spec.rb
    +++ b/spec/controllers/shop_controller_spec.rb
    @@ -54,7 +54,7 @@ describe ShopController do
         
    +    
    +    describe 'when a popularity of "1" is specified' do
    +      before :each do
    +        params[:popularity] = '1'
    +        controller.stubs(:find_xxxxxxs_from_temporary_table)
    +      end
    +      
    +      currently 'builds a table containing xxxxxxs which have sold well in the past week' do
    +        target = 
    +          %q{CREATE TEMPORARY TABLE available_xxxxxxs AS SELECT xxxxxxs.id, xxxxxxs.xxxxxx_id, xxxxxxs.user_account_id, } +
    +          %q{COALESCE( sales.xxxxxx_sales, 0 ) AS xxxxxx_sales, COALESCE( xxxxxx.newest_xxxxxx, '1970-01-01' ) AS newest_xxxxxx } +
    +          %q{FROM xxxxxxs INNER JOIN xxxxxxs ON xxxxxxs.xxxxxx_id = xxxxxxs.id LEFT JOIN ( SELECT xxxxxxs.xxxxxx_id, } +
    +          %q{SUM( shop_order_items.quantity ) AS xxxxxx_sales FROM shop_order_items INNER JOIN xxxxxxs ON } +
    +          %q{shop_order_items.xxxxxx_id = xxxxxxs.id AND shop_order_items.created_on >= current_date - 7 } +
    +          %q{GROUP BY xxxxxxs.xxxxxx_id ) AS sales ON xxxxxxs.xxxxxx_id = sales.xxxxxx_id LEFT JOIN } +
    +          %q{( SELECT xxxxxx_xxxxxx.xxxxxx_id, MAX( CASE xxxxxx_xxxxxx.xxxxxxning_date <= current_date WHEN } +
    +          %q{true THEN xxxxxx_xxxxxx.xxxxxxning_date ELSE NULL END ) AS newest_xxxxxx FROM xxxxxx_xxxxxx } +
    +          %q{GROUP BY xxxxxx_xxxxxx.xxxxxx_id ) AS xxxxxx ON xxxxxxs.id = xxxxxx.xxxxxx_id WHERE } +
    +          %q{xxxxxxs.active = 1 AND xxxxxxs.inventory_produced > xxxxxxs.inventory_sold AND xxxxxxs.out_of_stock = 0 } +
    +          %q{AND xxxxxxs.active = 1 AND xxxxxxs.browsable = 't' AND ( xxxxxx.xxxxxx_id IS NULL OR xxxxxx.newest_xxxxxx IS NOT NULL ) AND sales.xxxxxx_id IS NOT NULL}
    +        controller.expects(:build_temporary_available_xxxxxxs_table).with {|args| args.strip.split("\n").join(' ').gsub(/\s+/, ' ') ==  target }
    +        do_get
    +      end
    +    end
    
       # ...
    

!SLIDE code smallest

    @@@diff
    Author: rick
    
        Extracting SQL temp table methods.
        
        Now that we've proven that there are 5 (related, granted) methods for generating
        the temp table creation SQL we pull them out to partially clean up the method.
        
        When we've characterized the remainder of the method (especially the lookup-in-
        temp-table code) we can begin to migrate this functionality down to the Design
        model where it belongs, as named scopes / finder methods, and then this method
        will be trimmed down to almost nothing.  Then we can finally make it a before_filter
        instead of this bitched-up inline thing.
        
    --- a/app/controllers/shop_controller.rb
    +++ b/app/controllers/shop_controller.rb
    @@ -78,6 +78,61 @@ class ShopController < ApplicationController
       def find_designs_from_temporary_table(selector_sql, params)
         Design.select_values_with_sanitize( [selector_sql, params] ).map { |design_id| design_id.to_i }.uniq
       end
    +      
    +  def weekly_winners_sql
    +    %q{CREATE TEMPORARY TABLE available_products AS SELECT products.id, products.design_id, designs.user_account_id, } +
         # ...
    +  end
    +  
    +  def monthly_winners_sql
    +    %q{CREATE TEMPORARY TABLE available_products AS SELECT products.id, products.design_id, designs.user_account_id, } +
         # ...
    +  end
    +  
         # ...   

!SLIDE

    .... after 2 weeks of refactoring ...

!SLIDE code smallest
    @@@ruby
    def available_xxxxs_init
      params[:category] = parse_category(params[:category])
      
      popular_sales, popular_xxxx, weekly, monthly = parse_popularity(params[:popularity])
      set_params_to_i( [:popularity, :size, :artist, :color_category] ) 
      
      scope, conditions = {}, {}
      conditions.merge!('xxxxs.xxxx_category_id' => params[:category]) unless params[:category].blank?
      conditions.merge!('xxxxs.xxxx_size_id'     => params[:size])     unless params[:size].blank?
      conditions.merge!('xxxxs.user_account_id'      => params[:artist])   unless params[:artist].blank?
      conditions.merge!(ColorCategory.get_color_filter_from_color_category(params[:color_category])) unless params[:color_category].blank?
    
      scope.merge!(:conditions => conditions) unless conditions.blank?
      scope.merge!(:having => 'count(xxxxs.id) >= 3') if params[:size].blank?
      scope.merge!(Xxxxx.compute_ordering_for_lookup(params[:sort], popular_sales))
    
      all_available_xxxxs = Xxxxx.available_month_winners(scope)   if monthly && popular_xxxx
      all_available_xxxxs = Xxxxx.available_week_winners(scope)    if weekly  && popular_xxxx
      all_available_xxxxs = Xxxxx.available_monthly_sellers(scope) if monthly && popular_sales
      all_available_xxxxs = Xxxxx.available_weekly_sellers(scope)  if weekly  && popular_sales
      all_available_xxxxs = Xxxxx.available_xxxxs(scope)         if !popular_sales && !popular_xxxx
    
      @all_available_xxxxs = all_available_xxxxs.uniq
    end

!SLIDE 

    Using named scopes and 1-liner 
    helper methods on the model of
    interest.

!SLIDE

[ 300-500ms page load times on critical controller ]

!SLIDE

[ server load significantly reduced ]

!SLIDE 

[ improved code in tests in hot-spot areas ]