# ARMarkerless
Assigment

using System.Collections;
using UnityEngine;
using UnityEngine.UI;

public class arproject : MonoBehaviour {

	public Transform object1;
	public Transform object2;

	public GameObject text1;
	public GameObject text2;

	private float originalLatitude;
	private float originalLongitude;
	private float currentLongitude;
	private float currentLatitude;

	//private GameObject distanceTextObject;
	private double distance;

	private bool setOriginalValues = true;

	private Vector3 targetPosition1;
	private Vector3 targetPosition2;
	private Vector3 originalPosition1;
	private Vector3 originalPosition2;

	private float objek1Longitude = (float)112.618134;
	private float objek1Latitude = (float)-7.952185;

	//objek 1 -7.952185, 112.618134
	//objek 2 -7.954781, 112.615005

	private float objek2Longitude = (float)112.615005;
	private float objek2Latitude = (float)-7.954781;

	private float speed = .1f;

	IEnumerator GetCoordinates()
	{
		//while true so this function keeps running once started.
		while (true) {
			// check if user has location service enabled
			if (!Input.location.isEnabledByUser)
				yield break;

			// Start service before querying location
			Input.location.Start (1f,.1f);

			// Wait until service initializes
			int maxWait = 20;
			while (Input.location.status == LocationServiceStatus.Initializing && maxWait > 0) {
				yield return new WaitForSeconds (1);
				maxWait--;
			}

			// Service didn't initialize in 20 seconds
			if (maxWait < 1) {
				print ("Timed out");
				yield break;
			}

			// Connection has failed
			if (Input.location.status == LocationServiceStatus.Failed) {
				print ("Unable to determine device location");
				yield break;
			} else {
				// Access granted and location value could be retrieved
				print ("Location: " + Input.location.lastData.latitude + " " + Input.location.lastData.longitude + " " + Input.location.lastData.altitude + " " + Input.location.lastData.horizontalAccuracy + " " + Input.location.lastData.timestamp);

				//if original value has not yet been set save coordinates of player on app start
				if (setOriginalValues) {
					originalLatitude = Input.location.lastData.latitude;
					originalLongitude = Input.location.lastData.longitude;
					setOriginalValues = false;
				}

				//overwrite current lat and lon everytime
				currentLatitude = Input.location.lastData.latitude;
				currentLongitude = Input.location.lastData.longitude;

				//calculate the distance between where the player was when the app started and where they are now.
				//Calc (originalLatitude, originalLongitude, currentLatitude, currentLongitude);
				Calc (objek1Latitude, objek1Longitude, currentLatitude, currentLongitude, "objek1");
				Calc (objek2Latitude, objek2Longitude, currentLatitude, currentLongitude, "objek2");
			}
			Input.location.Stop();
		}
	}

	//calculates distance between two sets of coordinates, taking into account the curvature of the earth.
	public void Calc(float lat1, float lon1, float lat2, float lon2, string whichObject)
	{

		var R = 6378.137; // Radius of earth in KM
		var dLat = lat2 * Mathf.PI / 180 - lat1 * Mathf.PI / 180;
		var dLon = lon2 * Mathf.PI / 180 - lon1 * Mathf.PI / 180;
		float a = Mathf.Sin(dLat / 2) * Mathf.Sin(dLat / 2) +
			Mathf.Cos(lat1 * Mathf.PI / 180) * Mathf.Cos(lat2 * Mathf.PI / 180) *
			Mathf.Sin(dLon / 2) * Mathf.Sin(dLon / 2);
		var c = 2 * Mathf.Atan2(Mathf.Sqrt(a), Mathf.Sqrt(1 - a));
		distance = R * c;
		distance = distance * 1000f; // meters
		//set the distance text on the canvas
		//distanceTextObject.GetComponent<Text> ().text = "Distance: " + distance;

		//convert distance from double to float
		float distanceFloat = (float)distance;
		//set the target position of the ufo, this is where we lerp to in the update function
		//targetPosition = originalPosition - new Vector3 (0, 0, distanceFloat * -6);
		if(whichObject.Equals("objek1")){
			targetPosition1 = originalPosition1 + new Vector3 (0, 0, distanceFloat);
			text1.GetComponent<Text>().text = "Distance: " + distance;
		}
		if(whichObject.Equals("objek2")){
			targetPosition2 = originalPosition2 + new Vector3 (0, 0, distanceFloat);
			text2.GetComponent<Text>().text = "Distance: " + distance;
		}
		//distance was multiplied by 12 so I didn't have to walk that far to get the UFO to show up closer

	}

	void Start(){
		//get distance text reference
		//start GetCoordinate() function 
		StartCoroutine ("GetCoordinates");
		//initialize target and original position
		targetPosition1 = object1.position;
		originalPosition1 = object1.position;

		targetPosition2 = object2.position;
		originalPosition2 = object2.position;

	}

	void Update(){
		//linearly interpolate from current position to target position
		object1.position = Vector3.Lerp(object1.position, targetPosition1, speed);
		//rotate by 1 degree about the y axis every frame
		object1.eulerAngles += new Vector3 (0, 1f, 0);

		//linearly interpolate from current position to target position
		object2.position = Vector3.Lerp(object2.position, targetPosition2, speed);
		//rotate by 1 degree about the y axis every frame
		object2.eulerAngles += new Vector3 (0, 1f, 0);
	}
}
