
!SLIDE 	

    Removed an entire subsystem related 
    to A/B testing experiments, which was 
    adding overhead to every request.

    The "data" was never used.

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

    def available_products_init
     if params[:category]
       category_string = params[:category].to_s.strip.downcase
       params[:category] = case
                           when category_string == 'xxx' then '1'
                           when category_string == 'xxx' then '2'
                           when category_string.to_i > 0 then category_string.to_i.to_s
                           else nil
                           end
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
     xxxxxx_xxxxxx_sql += %q{
         GROUP BY xxxxxx_xxxxxx.xxxxxx_id
     }
     available_XXXXX_create = %q{
         CREATE TEMPORARY TABLE available_XXXXX AS
         SELECT XXXXX.id, XXXXX.xxxxxx_id, xxxxxxs.user_account_id, 
     }
     if popular_sales
       available_XXXXX_create += %q{
           COALESCE( sales.xxxxxx, 0 ) AS xxxxxx,
       }
     end
     available_XXXXX_create += %q{
           COALESCE( xxxxxx.newest_xxxxxx, '1970-01-01' ) AS newest_xxxxxx
         FROM XXXXX
           INNER JOIN xxxxxxs ON XXXXX.xxxxxx_id = xxxxxxs.id
     }
     if popular_sales
       available_XXXXX_create += %Q{
             LEFT JOIN (
               #{xxxxxx_sql}
              ) AS sales ON XXXXX.xxxxxx_id = sales.xxxxxx_id
       }
     end
     available_XXXXX_create += %q{
           LEFT JOIN (
     }
     available_XXXXX_create += xxxxxx_xxxxxx_sql
     available_XXXXX_create += %q{
           ) AS xxxxxx ON xxxxxxs.id = xxxxxx.xxxxxx_id
         WHERE XXXXX.active = 1
         AND XXXXX.inventory_produced > XXXXX.inventory_sold
         AND XXXXX.out_of_stock = 0
         AND xxxxxxs.active = 1
         AND xxxxxxs.browsable = 't'
         AND ( xxxxxx.xxxxxx_id IS NULL OR xxxxxx.newest_xxxxxx IS NOT NULL )
     }
     available_XXXXX_create += "AND sales.xxxxxx_id IS NOT NULL " if popular_sales
     available_XXXXX_create += "AND xxxxxx.xxxxxx_id IS NOT NULL " if popular_xxxxxxs
    
     set_params_to_i( [:popularity, :category, :size, :artist, :color_category] ) 
    
     @param_variables = params
     @per_param_conditions = {}
     @per_param_conditions[:category] = 'AND XXXXX.product_category_id = :category ' unless params[:category].blank?
     @per_param_conditions[:size] = 'AND XXXXX.product_size_id = :size ' unless params[:size].blank?
     @per_param_conditions[:artist] = 'AND available_XXXXX.user_account_id = :artist ' unless params[:artist].blank?
     if params[:color_category]
       color_category = COLOR_CATEGORIES.detect {|cc| cc[:id] == params[:color_category].to_i }
       if color_category
         @per_param_conditions[:color] = 'AND XXXXX.product_color_id IN ( ' + color_category[:colors].join( ', ' ) + ' ) '
       end
     end
    
     XXXXX.connection.execute( 'DROP TABLE IF EXISTS available_XXXXX' )
     XXXXX.connection.execute( available_XXXXX_create )
    
     available_xxxxxxs_sql = %q{
         SELECT available_XXXXX.xxxxxx_id AS id,
           XXXXX.product_category_id AS gender_id,
           AVG( XXXXX.price ) AS average_price,
           COUNT( XXXXX.id ) AS in_stock_size_count,
     }
     if popular_sales
       available_xxxxxxs_sql += %q{
             SUM( available_XXXXX.xxxxxx ) AS xxxxxx_sales,
       }
     end
     available_xxxxxxs_sql += %q{
           MAX( available_XXXXX.newest_xxxxxx ) AS newest_xxxxxx
         FROM available_XXXXX
           INNER JOIN XXXXX ON XXXXX.id = available_XXXXX.id
         WHERE true
      }
     @per_param_conditions.each_value {|condition| available_xxxxxxs_sql += condition }
     available_xxxxxxs_sql += 'GROUP BY available_XXXXX.xxxxxx_id, XXXXX.product_category_id '
     if params[:size].blank?
       available_xxxxxxs_sql += 'HAVING COUNT( XXXXX.id ) >= 3 '
     end
     if not params[:sort].blank?
       if params[:sort] == '2'
         available_xxxxxxs_sql += 'ORDER BY average_price DESC '
       else
         available_xxxxxxs_sql += 'ORDER BY average_price ASC '
       end
     elsif popular_sales
       available_xxxxxxs_sql += 'ORDER BY xxxxxx_sales DESC '
     else
       available_xxxxxxs_sql += 'ORDER BY newest_xxxxxx DESC '
     end
     @all_available_xxxxxx_ids = Design.select_values_with_sanitize( [available_xxxxxxs_sql, @param_variables] ).map { |xxxxxx_id| xxxxxx_id.to_i }.uniq
    end
    
    
    ove from sql to AR
    mpact: significant speedups, reduced server load
    