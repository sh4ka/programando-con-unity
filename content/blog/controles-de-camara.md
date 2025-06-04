+++
title = "Cómo mover la cámara con WASD y hacer zoom suave con la rueda del ratón en Unity"
date = 2025-06-04
+++

# Cómo mover la cámara con WASD y hacer zoom suave con la rueda del ratón en Unity

> En este mini tutorial aprenderás a mover la cámara en Unity usando las teclas **WASD** y a hacer **zoom con la rueda del ratón** de forma suave y natural, usando el nuevo **Input System** de Unity.

---

## Índice

1. [Preparar el proyecto](#1-preparar-el-proyecto)
2. [Script CameraController](#2-script-cameracontroller)
3. [Asignar el script a la cámara](#3-asignar-el-script-a-la-camara)
4. [Notas sobre Input System](#4-notas-sobre-input-system)

---

## 1. Preparar el proyecto

1. Abre Unity y crea un proyecto nuevo en 2D.
2. Asegúrate de tener instalado el paquete **Input System**:

   * Ve a **Window → Package Manager**.
   * Busca **Input System** e instálalo si no está.
3. En **Edit → Project Settings → Player → Other Settings**, asegúrate que **Active Input Handling** esté en "Input System Package (New)" o "Both".

---

## 2. Script CameraController

Crea un nuevo script C# llamado `CameraController.cs` dentro de `Assets/Scripts/` con el siguiente contenido:

```cs
using UnityEngine;
using UnityEngine.InputSystem;

public class CameraController : MonoBehaviour
{
    [Header("Movement Settings")]
    public float moveSpeed = 5f;
    public float acceleration = 10f;

    [Header("Zoom Settings")]
    public float zoomSpeed = 5f;
    public float minZoom = 3f;
    public float maxZoom = 15f;

    private Vector3 targetVelocity;
    private Vector3 currentVelocity;

    private Camera cam;

    void Start()
    {
        cam = Camera.main;
    }

    void Update()
    {
        HandleMovement();
        HandleZoom();
    }

    void HandleMovement()
    {
        Vector3 inputDirection = Vector3.zero;

        if (Keyboard.current.wKey.isPressed) inputDirection += Vector3.up;
        if (Keyboard.current.sKey.isPressed) inputDirection += Vector3.down;
        if (Keyboard.current.aKey.isPressed) inputDirection += Vector3.left;
        if (Keyboard.current.dKey.isPressed) inputDirection += Vector3.right;

        targetVelocity = inputDirection.normalized * moveSpeed;
        currentVelocity = Vector3.Lerp(currentVelocity, targetVelocity, acceleration * Time.deltaTime);
        transform.position += currentVelocity * Time.deltaTime;
    }

    void HandleZoom()
    {
        float scrollInput = Mouse.current.scroll.ReadValue().y;
        float targetSize = cam.orthographicSize - scrollInput * zoomSpeed * Time.deltaTime;
        cam.orthographicSize = Mathf.Lerp(cam.orthographicSize, targetSize, 0.1f);
        cam.orthographicSize = Mathf.Clamp(cam.orthographicSize, minZoom, maxZoom);
    }
}
```

---

## 3. Asignar el script a la cámara

1. Selecciona tu **Main Camera** en el panel `Hierarchy`.
2. Pulsa **Add Component** y selecciona `CameraController`.
3. Ajusta en el Inspector los parámetros si lo deseas:

   * `Move Speed`, `Zoom Speed`, `Min Zoom`, `Max Zoom`, etc.

---

## 4. Notas sobre Input System

Este script está hecho para funcionar con el **nuevo Input System**. Si en tu proyecto tienes errores como:

```
InvalidOperationException: You are trying to read Input using the UnityEngine.Input class, but you have switched active Input handling to Input System package in Player Settings.
```

Debes hacer una de estas dos cosas:

* O bien cambias en **Player Settings** el `Active Input Handling` a `Both` o `Input System Package (New)`.
* O bien adaptas el código para que use `Input.GetAxis` y `Input.mouseScrollDelta`, pero perderías la compatibilidad moderna.

---

Con eso tendrás una cámara que se mueve de forma natural con WASD y que hace zoom suave con la rueda del ratón. Ideal para juegos de estrategia, city builders o mundos abiertos 2D.

¡A explorar tu mundo con estilo! 🌍
