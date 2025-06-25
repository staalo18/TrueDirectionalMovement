Changelog:

Hooks.cpp and Hooks.h
* New class DragonCameraState, same functions and members as HorseCameraState, but for the dragon camerastate
* New hook DragonCameraStateHook (same functions as HorseCameraStateHook, but hooking into the DragonCameraState)
* Renamed SaveCamera::RotationType::kHorse to SaveCamera::RotationType::kMount to reflect both horse and dragon mounts (no functional changes)

DirectionalMovementHandler.cpp and Hooks.cpp
* Added checks for RE::CameraStates::kDragon whererever there are checks for RE::CameraState::kMount
* Use DragonCameraState instead of HorseCameraState in case camera state is RE::CameraState::kDragon

DirectionalMovementHandler.cpp
* modified handling of camera snap in ToggleTargetLock() and LookAtTarget() such that DragonCameraState->dragonRefHandle.get()->As<RE::Actor>()->data.angle.x is not modified.
( otherwise if target lock is on, the dragon's orientation flickers between two positions while the dragon is switching flying states (flying->hover, or hover->land)
* In ToggleTargetLock(), when toggling the TargetLock on, the target lock is set to the dragon's current combat target instead of using the actor which is closest to the center of the screen
 (only in case IDRC is active and the dragon is mounted and in combat). This functionality uses IDRC's API function APIs::IDRC->GetCurrentTarget().
* In LookAtTarget(): 
  - use dragon instead of player as reference because player's yaw is changing with dragon's head orientation.

  - enhanced the handling of the situation when the target is behind the camera (ie camera is between player and target):
    - added distance check to bIsBehind to avoid that the camera is rotating towards the player-axis in case the target is further away from the player as the camera
  (in that case, the target ended up behind the camera).  
    - The distance check also addresses this case: If the target is close behind the camera there was a tendency for camera oscillation between two positions close to the player-target axis. Reason is that projectedDirectionToTargetXY changes significantly through the camera movement because the distance btw camera and target is short. That in turn leads to angleDelta values switching sign between frames.

 * IDRC API is added via API/IDRC_API.h, and connected in API/APIManager.cpp and API/APIManager.h 
    * IDRC API requires unreleased version of IDRC (https://github.com/staalo18/IntuitiveDragonRideControl)

* New TDM settings (in Settings.h) - not in MCM:
    * bTargetLockOnIDRCTarget (ON): Use IDRC's current target when activating the TargetLock, instead of the actor closest to the center of the screen center
    * bEnableCameraBehindTarget (OFF): Enable camera rotation towards the target when the camera is behind the camera, ie target is between player and camera

* Changed the following TDM MCM settings during gameplay:
    * Disable Directional Movement  - Auto-adjust Camera when mounted on a dragon
    * Disable TargetLock - Check LOS when mounted on a dragon