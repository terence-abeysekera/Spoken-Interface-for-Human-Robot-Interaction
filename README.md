# Spoken-Interface-for-Human-Robot-Interaction
Voice Controlled Robot Movement Using a Spoken Interface in Unity.
/// Terence Abeysekera 
/// SRN:22066264
/// Voice Controlled Robot Movement Using a Spoken Interface in Unity.
/// ** Controls a GameObject (simulated robot) using voice commands via the
/// Windows KeywordRecognizer. Implements a scalable Dictionary based command pattern.**

using System;
using System.Collections.Generic;
using System.Linq;
using UnityEngine;
using UnityEngine.Windows.Speech;

public class VoiceController : MonoBehaviour // This essentially tells Unity: "This is a component (a 'Script') that can be attached to a virtual object (a 'robot') in the game world.

{
    
    /// *** Configuration Variables ***
    

    // Distance the object moves per command (in units).
    private float moveDistance = 1.0f;

    // Speed used for smooth linear interpolation (Lerp). Balances quick response with smooth movement.
    private float smoothSpeed = 5.0f;

    
    /// *** State Variables ***
    

    // Core ASR engine; handles mic input and speech-to-text conversion.
    private KeywordRecognizer keywordRecognizer;

    // Dictionary mapping recognized voice commands (string) to their execution methods (Action).
    private Dictionary<string, Action> actions = new Dictionary<string, Action>();

    // The intended destination point for smooth movement (Vector3).
    private Vector3 targetPosition;

    
    /// *** Lifecycle Methods ***
    


    // Called once at startup for initialization. Sets up commands and starts the recognizer.
    // In this method, will register our voice commands, set up the recognizer, and start listening for commands.
    void Start()
    {
        // Initializes targetPosition to the current position to prevent erratic jumps at startup.
        targetPosition = transform.position;

        // 1. Register Commands & Synonyms
        actions.Add("left", MoveLeft);
        actions.Add("right", MoveRight);
        actions.Add("up", MoveUp);
        actions.Add("down", MoveDown);
        actions.Add("go left", MoveLeft);
        actions.Add("go right", MoveRight);
        actions.Add("move up", MoveUp);
        actions.Add("back", MoveDown);

        // 2. Initialize and Start Recognizer
        // Loads command keywords into the ASR engine and starts listening.

        keywordRecognizer = new KeywordRecognizer(actions.Keys.ToArray());
        keywordRecognizer.OnPhraseRecognized += OnPhraseRecognized;
        keywordRecognizer.Start();

        Debug.Log("Voice Control Initialized. Listening for: " + string.Join(", ", actions.Keys));
    }

    
    // Called every frame for continuous processes. Manages smooth movement.
  
    void Update()
    {
        // Smoothly interpolates the object's current position toward the target position.
        // This ensures fluid motion (HRI Design Principle) by avoiding instant jumps.
        transform.position = Vector3.Lerp(transform.position, targetPosition, Time.deltaTime * smoothSpeed);
    }

    /// *** Event Handler (Input Processing) ***

    
    // Event handler automatically called when a voice command is recognized.
    
    private void OnPhraseRecognized(PhraseRecognizedEventArgs args)
    {
        Debug.Log($"Command Detected: '{args.text}' with Confidence: {args.confidence}"); // logging statement used for debugging and monitoring the spoken interface while it is running.

        // 1. Validation and Lookup: Checks if the recognized text is a valid, mapped command.
        if (actions.ContainsKey(args.text))
        {
            // 2. Execution: Invokes the method associated with the command string.
            actions[args.text].Invoke();
        }
    }

    
    /// *** Action Methods (Robot Behavior) ***
    

    // Calculates the new target position for left movement (negative X-axis).
    private void MoveLeft()
    {
        targetPosition += Vector3.left * moveDistance;
    }

    // Calculates the new target position for right movement (positive X-axis).
    private void MoveRight()
    {
        targetPosition += Vector3.right * moveDistance;
    }

    // Calculates the new target position for forward movement (positive Z-axis).
    private void MoveUp()
    {
        targetPosition += Vector3.forward * moveDistance;
    }

    // Calculates the new target position for backward movement (negative Z-axis).
    private void MoveDown()
    {
        targetPosition += Vector3.back * moveDistance;
    }

    /// *** End Proces ***

    
    // Called when the GameObject is destroyed. Stops the recognizer and releases the microphone.
    
    private void OnDestroy()
    {
        // Stops and disposes of the KeywordRecognizer to release system resources (microphone handle).
        if (keywordRecognizer != null && keywordRecognizer.IsRunning)
        {
            keywordRecognizer.Stop();
            keywordRecognizer.Dispose();
        }
    }
}
