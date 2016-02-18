# PickCrop
Pickcrop provides a simple way yo picking an image from documents (either camera, gallery, Photos, Drive) and edit(Scale, Crop, Orientation, Zoom) it.

Library used : [uCrop](https://github.com/Yalantis/uCrop)


# How to use:

## Section A : Themeing
Copy these lines in **styles.xml** along with your theme. 
```java
<style name="NoActionBarAppTheme" parent="Theme.AppCompat.Light.NoActionBar"/>
```
## Section B : AndroidManifest.xml
Copy these lines before application tag : 
```java
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```
Declare this activity within application tag : 
```java
<activity android:name="com.yalantis.ucrop.UCropActivity" android:theme="@style/NoActionBarAppTheme" android:screenOrientation="portrait"/>    
```
## Section C: Gradle
Go to your module's build.gradle and add this : 
```java
repositories {
    maven {
        url 'https://dl.bintray.com/devdigitalappteam/maven/'
    }
}
```
Add this in dependancies : 
```java
compile 'com.devdigital:pickcrop:1.0'
```
## Section D: In code
Declaration
```java
public class MainActivity extends AppCompatActivity{
    private ImageHelper imageHelper;
    private UCropHelper uCropHelper;
}
```
Initialization
```java
public class MainActivity extends AppCompatActivity implments UCropHelper.UCropImageCallback, ImageHelper.RuntimePermissionCallback{
    private ImageHelper imageHelper;
    private UCropHelper uCropHelper;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        imageHelper = new ImageHelper(this, this);
        uCropHelper = new UCropHelper(this, this);
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        ImageHelper.onDestroy();
    }
}
```
Use image picking from Camera or Documents(Gallery)
```java
public class MainActivity extends AppCompatActivity implments UCropHelper.UCropImageCallback, ImageHelper.RuntimePermissionCallback{
    private ImageHelper imageHelper;
    private UCropHelper uCropHelper;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        imageHelper = new ImageHelper(this, this);
        uCropHelper = new UCropHelper(this, this);
		imageHelper.pick(true); // true for picking image from camera, else false for taking picture from gallery
    }
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        imageHelper.handleActivityResult(requestCode, resultCode, data, this, new DefaultCallback() {
            @Override
            public void onImagePickerError(Exception e, GlibImage.ImageSource source, int type) {
                //Some error handling
            }
            @Override
            public void onImagePicked(File imageFile, GlibImage.ImageSource source, int type) {
                // Handle the image
                // imagefile is the location of your image. 
                // for example 
                /*Picasso.with(this)
                .load(photoUri)
                .fit()
                .centerCrop()
                .into(imageView);*/
                // or Pass image for cropping
                uCropHelper.startCropping(Uri.fromFile(imageFile));
            }
            @Override
            public void onCanceled(GlibImage.ImageSource source, int type) {
                //Cancel handling, you might wanna remove taken photo if it was canceled
                if (source == GlibImage.ImageSource.CAMERA) {
                    File photoFile = GlibImage.lastlyTakenButCanceledPhoto(MainActivity.this);
                    if (photoFile != null) photoFile.delete();
                }
            }
        });
        // If image cropping has been selected in above code the need to use this for cropping image. 
        uCropHelper.handleActivityResult(requestCode, resultCode, data);
    }
    /**
    * We need to use this for handling runtime permission. 
    */
    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        if(requestCode == ImageHelper.REQUEST_WRITE_EXTERNAL_STORAGE){
            imageHelper.handleRequestPermissionsResult(requestCode, permissions, grantResults);
        } else {
            super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        }
    }
    @Override
    public void onPermissionGranted(boolean isCamera) {
        if (isCamera){
            ImageHelper.onTakePhotoClick();
        }else{
            ImageHelper.onPickFromGaleryClicked();
        }
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        ImageHelper.onDestroy();
    }   
}
```
