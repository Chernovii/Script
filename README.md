using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ControladorPersonaje : MonoBehaviour
{

public ControlNormal controlNormal;

public GameObject disparo;

public ScriptLaser scriptLaser;

//MOVIMIENTO HORIZONTAL VARIABLES
public int velocidadCarrera; //Velocidad a la que se puede mover
public int velocidadMaxima; //Velocidad a la que puede llegar
public int velocidadActual; //Registra la velocidad que tenemos

//SALTO VARIABLES
public float potenciaSalto; //Define cuanto puede saltar
public float velocidadVertical; //Almacena la velocidad vertical actual
public float velocidadCaida; //Velocidad a la que caes
public float velocidadCaidaMaxima; //Velocidad maxima a la que puede caer

public bool tocandoSuelo; //Detecta si estamos tocando suelo o no
public bool tieneSalto; //Detecta si hemos gastado el salto o no

//DISPARO VARIABLES
public bool disparoCargado; //Guarda si el disparo esta cargado o no
public float cadenciaDisparo; //Cuan rapido podemos disparar
public float recargaDisparo;
public Transform finalCuerno;

//RIGIDBODY
public Rigidbody rb;

//CAMARA

private Camera camara;
public float cameraVerticalAngle;
public float mouseSensitivity;
private float lastX;
private float lastY;




private void Awake()
{

    controlNormal = new ControlNormal();
    controlNormal.Enable();

}

void Start()
{
    camara = gameObject.GetComponentInChildren<Camera>();
}

// Update is called once per frame
void Update()
{
    //MOVIMIENTO GENERAL

    // - FRONTAL

    Vector3 velocidad = Vector3.zero;

    velocidad = camara.transform.forward * velocidadCarrera * controlNormal.personaje.DesplazamientoF.ReadValue<float>() +
    camara.transform.right * velocidadCarrera * controlNormal.personaje.DesplazamientoL.ReadValue<float>();

    velocidad.y = 0f;

    velocidad.x = Mathf.Clamp(velocidad.x, -velocidadMaxima, velocidadMaxima);
    velocidad.z = Mathf.Clamp(velocidad.z, -velocidadMaxima, velocidadMaxima);

    transform.position = transform.position + (velocidad / 100);


    // - SALTO


    velocidad.y = velocidadVertical;

    if (tocandoSuelo == true)
    {
        velocidad.y = 0;

        if (controlNormal.personaje.Salto.ReadValue<float>() > 0f)
        {
            velocidadVertical = potenciaSalto;
            transform.position = transform.position + (velocidad / 100);
            tocandoSuelo = false;
        }
    }

    if (tocandoSuelo == false)
    {

        velocidadVertical = velocidadVertical + velocidadCaida * Time.deltaTime;

        transform.position = transform.position + (velocidad / 100);
    }






    //Activamos la funcion de disparar

    HandleDisplacement();
    HandleCharacterLook();

    //Disparo();

    

}


private void OnTriggerEnter(Collider other)
{
    tocandoSuelo = true;
}

private void OnTriggerExit(Collider other)
{
    tocandoSuelo = false;
}

// - FUNCIONES CAMARA

private void HandleDisplacement()
{
    Vector3 velocidadCamara =

    controlNormal.personaje.CamaraHorizontal.ReadValue<float>() * camara.transform.right +
    controlNormal.personaje.CamaraVertical.ReadValue<float>() * camara.transform.forward;
}


private void HandleCharacterLook()
{
    float lookX = controlNormal.personaje.CamaraHorizontal.ReadValue<float>() - lastX;
    float lookY = controlNormal.personaje.CamaraVertical.ReadValue<float>() - lastY;
    transform.Rotate(new Vector3(0f, lookX * mouseSensitivity, 0f), Space.Self);
    cameraVerticalAngle -= lookY * mouseSensitivity;
    cameraVerticalAngle = Mathf.Clamp(cameraVerticalAngle, -89f, 89f);
    camara.transform.localEulerAngles = new Vector3(cameraVerticalAngle, 0f, 0f);

    lastX = controlNormal.personaje.CamaraHorizontal.ReadValue<float>();
    lastY = controlNormal.personaje.CamaraVertical.ReadValue<float>();

}
