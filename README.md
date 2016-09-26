# Parse-Upload-Image

##1.  Call to Upload user image after user profile updated
            if let currentUser = PF_User.currentUser(){
                userSvc.updateUserProfileInfo(currentUser, email: email.text!, name: name.text!, location: location.text!, timezone: timezonData[selectedTimezone!], completionHandler: { (success) in
                    if success{
                       
                         // Save profile image
                         
                        // To avoid image rotation issue.
                        let orientationFixedImage = self.imageProfile.image!.fixOrientation()
                        
                        userSvc.UploadUserImage(PF_User.currentUser()!,image: orientationFixedImage ) { (success) in
                            if success{
                                print("Image is uploaded sucessfully")
                                self.showAlert("Your profile is updated sucessfully.", message: "")

                            }else{
                                print("Error!!")
                                self.showAlert("Error occured while uploading image.", message: "")
                            }
                        }
                    }
                    
                })
            }

##2. Fix Orientation to avoid rotatino issue


import UIKit
extension UIImage {
    func fixOrientation() -> UIImage {
        if self.imageOrientation == UIImageOrientation.Up {
            return self
        }
        
        UIGraphicsBeginImageContextWithOptions(self.size, false, self.scale)
        self.drawInRect(CGRectMake(0, 0, self.size.width, self.size.height))
        let normalizedImage:UIImage = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        
        return normalizedImage;
    }
}


##3. upload image to parse 

func UploadUserImage(user : PF_User, image: UIImage?, completionHandler: (sucess: Bool) -> ()) {
        
        //create an image data
        
        //let imageData = UIImagePNGRepresentation(image!)
        // Small size representation
        let imageData = UIImageJPEGRepresentation(image!, 0.5)
        
        //create a parse file to store in cloud
        if let parseImageFile = PFFile(name: "uploaded_image.png", data: imageData!){
            parseImageFile.saveInBackgroundWithBlock({ (success, error) -> Void in
                if success{
                    user.Image = parseImageFile
                    user.saveInBackgroundWithBlock({ (success: Bool, error: NSError?) -> Void in
                        if error == nil {
                            print("Image is uploaded sucessfully")
                            completionHandler(sucess: true)
                        }else {
                            print(error)
                            completionHandler(sucess: false)
                        }
                    })
                }
            })
        }else{
            completionHandler(sucess: false)
        }

    }
