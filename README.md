using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Fox : MonoBehaviour
{
    //pubilc Fields
    public float speed = 1;


    //Private Fields
    Rigidbody2D rb;
    Animator animator;
    [SerializeField] Collider2D standingCollider;
    [SerializeField] Transform groundCheckCollider;
    [SerializeField] Transform overheadCheckCollider;
    [SerializeField] LayerMask groundLayer;
    [SerializeField] float jumpPower = 500;


    const float groundCheckRadius = 0.2f;
    const float overheadCheckRadius = 0.2f;
    float horizontalValue;
    float runSpeedModifier = 2f;
    float crouchSpeedModifier = 0.5f;

    bool isGrounded;
    bool isRunning = false;
    bool facingRight = true;
    bool jump;
    bool crouchPressed;

    void Awake()
    {
        rb = GetComponent<Rigidbody2D>();
        animator = GetComponent<Animator>();
    }
    
    void Update()
    {
        //Store the horizontal value
        horizontalValue = Input.GetAxisRaw("Horizontal");

        //If LShift is clicked enable isRunning
        if (Input.GetKeyDown(KeyCode.LeftShift))
            isRunning = true;

        //If LShift is released disable isRunning
        if (Input.GetKeyUp(KeyCode.LeftShift))
            isRunning = false;
        //If we press Jump button enable jump
        if (Input.GetButtonDown("Jump"))
            jump = true;
        //Otherwise disable it
        else if (Input.GetButtonUp("Jump"))
            jump = false;


        //If we press Crouch button enable crouch
        if (Input.GetButtonDown("Crouch"))
            crouchPressed = true;
        //Otherwise disable it
        else if (Input.GetButtonUp("Crouch"))
            crouchPressed = false;

    }
    void FixedUpdate()
    {
        GroundCheck();
        Move(horizontalValue, jump, crouchPressed);
    }

    void GroundCheck()
    {
        isGrounded = false;
        //Check if the GroundCheckObject is colling with other
        //2D Collider thet are in the "Ground" Layer
        //If yes (isGrounded true) else (isGrounded false)
        Collider2D[] colliders = Physics2D.OverlapCircleAll(groundCheckCollider.position, groundCheckRadius, groundLayer);
        if (colliders.Length > 0)
            isGrounded = true;
    }

    

    void Move(float dir ,bool jumpFlag,bool crouchFlag)
    {
        #region Jump && Crouch
        
        //If we are crouching and disabled crouching
        //Check overhead for collisions with Ground items
        //If there are any, remain crouched, otherwise un-crouch
        if(!crouchFlag)
        {
            if (Physics2D.OverlapCircle(overheadCheckCollider.position, overheadCheckRadius, groundLayer))
                crouchFlag = true;
        }
      

        //If we press Crouch we disable the standing collider + animate crouching
        //Reduce the speed
        //if released resume the orginal speed +
        //enable the standing collider + disable crouch animation
        if (isGrounded)
        {
            if(crouchFlag)
            {
                standingCollider.enabled = !crouchFlag;
            }
        
            //If yhe player is grounded and pressed space Jump
            if (jumpFlag)
            {
                isGrounded = false;
                jumpFlag = false;
                //Add jump force
                rb.AddForce(new Vector2(0f, jumpPower));
            }
        }

        animator.SetBool("Crouch", crouchFlag);
        #endregion

        #region Move & Run
        //Set value of x using dir and speed
        float xVal = dir * speed * 100 * Time.fixedDeltaTime;

        //If we are running multiply with the running modifier
        if (isRunning)
            xVal *= runSpeedModifier;

        //If we are running multiply with the running modifier
        if (crouchFlag)
            xVal *= crouchSpeedModifier;

        //Create Vec2 for the velocity
        Vector2 targetVelocity = new Vector2(xVal, rb.velocity.y);
        //Set the player's velocity
        rb.velocity = targetVelocity;

       
        //If looking right and clicked left (flip to the left)
        if(facingRight && dir<0)
        {
            transform.localScale = new Vector3(-1, 1, 1);
            facingRight = false;
        }
        //If looking left and clicked right (flip to the right)
        else if (!facingRight && dir > 0)
        {
            transform.localScale = new Vector3(1, 1, 1);
            facingRight = true;
        }

        // 0 idle , 4 walking , 8 running
        //Set the float xVelocity according to the x value
        //of the RigidBody2d velocity
        animator.SetFloat("xVelocity", Mathf.Abs(rb.velocity.x));
        #endregion
    }
}
