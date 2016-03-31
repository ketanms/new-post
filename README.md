function create_posttype() {
    register_post_type( 'Team',
        array(
            'labels' => array(
                'name' => 'Team',
                'singular_name' => 'Team Review',
                'add_new' => 'Add New',
                'add_new_item' => 'Add New Team',
                'edit' => 'Edit',
                'edit_item' => 'Edit Team',
                'new_item' => 'New Team',
                'view' => 'View',
                'view_item' => 'View Team',
                'search_items' => 'Search Team',
                'not_found' => 'No Team found',
                'not_found_in_trash' => 'No Team found in Trash',
                'parent' => 'Parent Team'
            ),
 
            'public' => true,
            'menu_position' => 15,
            'supports' => array( 'title', 'editor','thumbnail', 'custom-fields' ),
            'taxonomies' => array( '' ),
			'register_meta_box_cb' => 'team_price_box',
            'has_archive' => true
        )
    );
}

add_action( 'init', 'create_posttype' );


function wpshed_get_custom_field( $value ) {
	global $post;

    $custom_field = get_post_meta( $post->ID, $value, true );
    if ( !empty( $custom_field ) )
	    return is_array( $custom_field ) ? stripslashes_deep( $custom_field ) : stripslashes( wp_kses_decode_entities( $custom_field ) );

    return false;
}


/**
 * Register the Meta box
 */
 

add_action( 'init', 'create_posttype' );


add_action( 'add_meta_boxes', 'team_price_box' );
function team_price_box() {
    add_meta_box( 
        'team_price_box',
        __( 'Team Meta Box', 'myplugin_textdomain' ),
        'wpshed_meta_box_output',
        'team',
        'side',
        'high'
    );
	
}


/**
 * Output the Meta box
 */
function wpshed_meta_box_output( $post ) {
	// create a nonce field
	wp_nonce_field( 'my_wpshed_meta_box_nonce', 'wpshed_meta_box_nonce' ); ?>
	
    <p>
      <label for="users_login"><?php _e( 'UserName:', 'wpshed' ); ?>:</label>
        <select name="users_login" id="users_login" value="<?php echo wpshed_get_custom_field( 'users_login' ); ?>" />
              <?php
               $users = get_users();
               foreach($users as $user) { ?>
            <option value=<?php echo $user->data->ID; ?>><?php echo $user->data->user_login; ?></option>
        
      			<?php } ?>
       </select>
   
    </p>
	<p>
		<label for="team_areasize"><?php _e( 'Area Size', 'wpshed' ); ?>:</label>
		<input type="text" name="team_areasize" id="team_areasize" value="<?php echo wpshed_get_custom_field( 'team_areasize' ); ?>" size="25" />
    </p>
	
	<p>
		<label for="area_interest"><?php _e( 'Area Interest', 'wpshed' ); ?>:</label><br />
        <input type="checkbox" name="area_interest[]"  id="area_interest" value="development" />development<br/>
        <input type="checkbox" name="area_interest[]" id="area_interest" value="Designer" />Designer<br/>
        <input type="checkbox" name="area_interest[]" id="area_interest" value="Programmer" />Programmer<br/>
        <input type="checkbox" name="area_interest[]" id="area_interest" value="Testing" />Testing<br/>
    
    </p>
    
	<?php
}


/**
 * Save the Meta box values
 */
function wpshed_meta_box_save( $post_id ) {
	// Stop the script when doing autosave
	
	
	
	if( defined( 'DOING_AUTOSAVE' ) && DOING_AUTOSAVE ) return;

	// Verify the nonce. If insn't there, stop the script
	if( !isset( $_POST['wpshed_meta_box_nonce'] ) || !wp_verify_nonce( $_POST['wpshed_meta_box_nonce'], 'my_wpshed_meta_box_nonce' ) ) return;

	// Stop the script if the user does not have edit permissions
	if( !current_user_can( 'edit_post', get_the_id() ) ) return;

	// Save the textfield
		if( isset( $_POST['users_login'] ) )
			update_post_meta( $post_id, 'users_login', esc_attr( $_POST['users_login'] ) );
    // Save the textfield
	if( isset( $_POST['team_areasize'] ) )
		update_post_meta( $post_id, 'team_areasize', esc_attr( $_POST['team_areasize'] ) );
    	
	// Save the textarea
	$area_interest = implode(",",$_POST['area_interest']);
	
		update_post_meta( $post_id,'area_interest', esc_attr( $area_interest ) );	
}
add_action( 'save_post', 'wpshed_meta_box_save' );


function custom_metaboxes() {
	    add_meta_box('gallery_shortcode_info', 'Shortcode info', 'gallery_shortcode_fn1', 'team', 'side');
}
add_action('add_meta_boxes','custom_metaboxes');

function gallery_shortcode_fn1()
{
	global $post;
	
?>
	<input type="text" name="shrtcode" value="[TestShortcode id='<?php echo $post->ID; ?>']" readonly="readonly"/>
    
<?php 
		
}



    add_shortcode( 'TestShortcode', 'display_custom_post_type' );

    function display_custom_post_type(){
        $args = array(
            'post_type' => 'team',
			'posts_per_page' =>1,
            'post_status' => 'publish'
			
        );

        
        $query = new WP_Query( $args );
        if( $query->have_posts() ){
           
            while( $query->have_posts() ){
                $query->the_post();
                 echo '<span><b>Title:</b></span>'; 
				 echo the_title(); 
				 echo '</br/>';
				 echo '<span><b>Description:</b></span>'; 
				 echo the_content();
				 echo '<b>UserName:</b>';
				 $userdata = get_post_meta( get_the_ID(), 'users_login', true );
				 $userinfo = get_userdata($userdata);
				 echo $userinfo->user_login;
				 echo '<br><b>Area Size:</b>';
				 echo get_post_meta( get_the_ID(), 'team_areasize', true );
				 echo '<br><b>Area Interest:</b>';
				 echo get_post_meta( get_the_ID(), 'area_interest', true );
            }
           
        }
        wp_reset_postdata();
      }


