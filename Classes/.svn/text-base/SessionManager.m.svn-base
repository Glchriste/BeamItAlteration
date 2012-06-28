/*
 
 File: SessionManager.m
 Abstract: Delegate for the session and sends notifications when it changes.
 Version: 1.0
 
 Disclaimer: IMPORTANT:  This ArcTouch software is supplied to you by 
 ArcTouch Inc. ("ArcTouch") in consideration of your agreement to the 
 following terms, and your use, installation, modification or redistribution 
 of this ArcTouch software constitutes acceptance of these terms.  
 If you do not agree with these terms, please do not use, install, 
 modify or redistribute this ArcTouch software.
 
 In consideration of your agreement to abide by the following terms, and
 subject to these terms, ArcTouch grants you a personal, non-exclusive
 license, under ArcTouch's copyrights in this original ArcTouch software (the
 "ArcTouch Software"), to use, reproduce, modify and redistribute the ArcTouch
 Software, with or without modifications, in source and/or binary forms;
 provided that if you redistribute the ArcTouch Software in its entirety and
 without modifications, you must retain this notice and the following
 text and disclaimers in all such redistributions of the ArcTouch Software.
 Neither the name, trademarks, service marks or logos of ArcTouch Inc. may
 be used to endorse or promote products derived from the ArcTouch Software
 without specific prior written permission from ArcTouch.  Except as
 expressly stated in this notice, no other rights or licenses, express or
 implied, are granted by ArcTouch herein, including but not limited to any
 patent rights that may be infringed by your derivative works or by other
 works in which the ArcTouch Software may be incorporated.
 
 The ArcTouch Software is provided by ArcTouch on an "AS IS" basis.  ARCTOUCH
 MAKES NO WARRANTIES, EXPRESS OR IMPLIED, INCLUDING WITHOUT LIMITATION
 THE IMPLIED WARRANTIES OF NON-INFRINGEMENT, MERCHANTABILITY AND FITNESS
 FOR A PARTICULAR PURPOSE, REGARDING THE ARCTOUCH SOFTWARE OR ITS USE AND
 OPERATION ALONE OR IN COMBINATION WITH YOUR PRODUCTS.
 
 IN NO EVENT SHALL ARCTOUCH BE LIABLE FOR ANY SPECIAL, INDIRECT, INCIDENTAL
 OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 INTERRUPTION) ARISING IN ANY WAY OUT OF THE USE, REPRODUCTION,
 MODIFICATION AND/OR DISTRIBUTION OF THE ARCTOUCH SOFTWARE, HOWEVER CAUSED
 AND WHETHER UNDER THEORY OF CONTRACT, TORT (INCLUDING NEGLIGENCE),
 STRICT LIABILITY OR OTHERWISE, EVEN IF ARCTOUCH HAS BEEN ADVISED OF THE
 POSSIBILITY OF SUCH DAMAGE.
 
 Copyright (C) 2009 ArcTouch Inc. All Rights Reserved.
 */

#import "SessionManager.h"

@interface SessionManager ()

- (Device *)addDevice:(NSString *)peerID;
- (void)removeDevice:(Device *)device;
- (NSDictionary *)getDeviceInfo:(Device *)device;

@end


@implementation SessionManager

- (id)initWithDataHandler:(DataHandler *)handler devicesManager:(DevicesManager *)manager {
	self = [super init];
	
	if (self) {
		devicesManager = manager;

		beamItSession = [[GKSession alloc] initWithSessionID:BEAM_IT_GLOBAL_SESSION_ID displayName:nil sessionMode:GKSessionModePeer];
		beamItSession.delegate = self;
		[beamItSession setDataReceiveHandler:handler withContext:nil];
	}
	
	return self;
}

- (void)start {
	beamItSession.available = YES;
}

- (void)session:(GKSession *)session peer:(NSString *)peerID didChangeState:(GKPeerConnectionState)state {
	Device *currentDevice = [devicesManager deviceWithID:peerID];
	
	// Instead of trying to respond to the event directly, it delegates the events.
	// The availability is checked by the main ViewController.
	// The connection is verified by each Device.
	switch (state) {
		case GKPeerStateConnected:
			if (currentDevice) {
				[[NSNotificationCenter defaultCenter] postNotificationName:NOTIFICATION_DEVICE_CONNECTED object:nil userInfo:[self getDeviceInfo:currentDevice]];
			}
			break;
		case GKPeerStateConnecting:
		case GKPeerStateAvailable:
			if (!currentDevice) {
				currentDevice = [self addDevice:peerID];
				[[NSNotificationCenter defaultCenter] postNotificationName:NOTIFICATION_DEVICE_AVAILABLE object:nil userInfo:[self getDeviceInfo:currentDevice]];
			}
			break;
		case GKPeerStateUnavailable:
			if (currentDevice) {
				[currentDevice retain];
				[self removeDevice:currentDevice];
				[[NSNotificationCenter defaultCenter] postNotificationName:NOTIFICATION_DEVICE_UNAVAILABLE object:nil userInfo:[self getDeviceInfo:currentDevice]];
				[currentDevice release];
			}
			break;
		case GKPeerStateDisconnected:
			if (currentDevice) {
				[[NSNotificationCenter defaultCenter] postNotificationName:NOTIFICATION_DEVICE_DISCONNECTED object:nil userInfo:[self getDeviceInfo:currentDevice]];
			}
			break;
	}
}

- (Device *)addDevice:(NSString *)peerID {
	Device *device = [[Device alloc] initWithSession:beamItSession peer:peerID];
	[devicesManager addDevice:device];
	[device release];
	
	return device;
}

- (void)removeDevice:(Device *)device {
	[devicesManager removeDevice:device];
}

- (NSDictionary *)getDeviceInfo:(Device *)device {
	return [NSDictionary dictionaryWithObject:device forKey:DEVICE_KEY];
}

- (void)session:(GKSession *)session didReceiveConnectionRequestFromPeer:(NSString *)peerID {
	[beamItSession acceptConnectionFromPeer:peerID error:nil];
}

- (void)session:(GKSession *)session connectionWithPeerFailed:(NSString *)peerID withError:(NSError *)error {
	Device *currentDevice = [devicesManager deviceWithID:peerID];
	
	// Does the same thing as the didStateChange method. It tells a Device that the connection failed.
	if (currentDevice) {
		[[NSNotificationCenter defaultCenter] postNotificationName:NOTIFICATION_DEVICE_CONNECTION_FAILED object:nil userInfo:[self getDeviceInfo:currentDevice]];
	}
}

- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex {
	exit(0);
}

- (void)session:(GKSession *)session didFailWithError:(NSError *)error {
	UIAlertView *errorView = [[UIAlertView alloc] initWithTitle:NSLocalizedString(@"BLUETOOTH_ERROR_TITLE", @"Title for the error dialog.")
														message:NSLocalizedString(@"BLUETOOTH_ERROR", @"Wasn't able to make bluetooth available")
													   delegate:self
											  cancelButtonTitle:@"OK"
											  otherButtonTitles:nil];
	
	[errorView show];
	[errorView release];
}

@end
