# tukumon
まがしゅご！つくもんのプログラム
using UnityEngine;

public class SimpleFPSController : MonoBehaviour
{
    [Header("References")]
    public CharacterController controller;
    public Transform cameraTransform;

    [Header("Movement")]
    public float moveSpeed = 4f;
    public float gravity = -20f;

    [Header("Look")]
    public float mouseSensitivity = 100f;
    public float maxLookAngle = 80f;

    [Header("Head Bob")]
    public float bobFrequency = 10f;   // 흔들리는 속도
    public float bobAmount = 0.05f;    // 흔들리는 높이
    public float returnSpeed = 8f;     // 멈췄을 때 원래 위치로 돌아가는 속도

    private float xRotation = 0f;
    private float verticalVelocity = 0f;

    private Vector3 cameraDefaultLocalPos;
    private float bobTimer = 0f;

    void Start()
    {
        Cursor.lockState = CursorLockMode.Locked;
        Cursor.visible = false;

        if (controller == null)
            controller = GetComponent<CharacterController>();

        cameraDefaultLocalPos = cameraTransform.localPosition;
    }

    void Update()
    {
        Look();
        Move();
        HandleHeadBob();
    }

    void Look()
    {
        float mouseX = Input.GetAxis("Mouse X") * mouseSensitivity * Time.deltaTime;
        float mouseY = Input.GetAxis("Mouse Y") * mouseSensitivity * Time.deltaTime;

        xRotation -= mouseY;
        xRotation = Mathf.Clamp(xRotation, -maxLookAngle, maxLookAngle);

        cameraTransform.localRotation = Quaternion.Euler(xRotation, 0f, 0f);
        transform.Rotate(Vector3.up * mouseX);
    }

    void Move()
    {
        float x = Input.GetAxis("Horizontal");
        float z = Input.GetAxis("Vertical");

        Vector3 move = transform.right * x + transform.forward * z;
        controller.Move(move * moveSpeed * Time.deltaTime);

        if (controller.isGrounded && verticalVelocity < 0)
        {
            verticalVelocity = -2f;
        }

        verticalVelocity += gravity * Time.deltaTime;
        Vector3 gravityMove = new Vector3(0f, verticalVelocity, 0f);
        controller.Move(gravityMove * Time.deltaTime);
    }

    void HandleHeadBob()
    {
        float x = Input.GetAxis("Horizontal");
        float z = Input.GetAxis("Vertical");

        bool isMoving = Mathf.Abs(x) > 0.1f || Mathf.Abs(z) > 0.1f;

        if (controller.isGrounded && isMoving)
        {
            bobTimer += Time.deltaTime * bobFrequency;

            float bobOffsetY = Mathf.Sin(bobTimer) * bobAmount;
            float bobOffsetX = Mathf.Sin(bobTimer * 0.5f) * (bobAmount * 0.5f);

            cameraTransform.localPosition = cameraDefaultLocalPos + new Vector3(bobOffsetX, bobOffsetY, 0f);
        }
        else
        {
            bobTimer = 0f;
            cameraTransform.localPosition = Vector3.Lerp(
                cameraTransform.localPosition,
                cameraDefaultLocalPos,
                Time.deltaTime * returnSpeed
            );
        }
    }
}
