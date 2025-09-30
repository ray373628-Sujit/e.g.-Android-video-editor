
AndroidVideoEditor/
├── .gitignore
├── README.md
├── app/
│   ├── build.gradle
│   └── src/
│       └── main/
│           ├── AndroidManifest.xml
│           ├── java/
│           │   └── com/example/androidvideoeditor/
│           │       ├── MainActivity.java
│           │       └── VideoUtils.java
│           └── res/
│               ├── layout/
│               │   └── activity_main.xml
│               └── values/
│                   └── strings.xml
└── build.gradle
```

---

## 📝 File Contents

### `.gitignore`

```
*.iml
.gradle/
/local.properties
/.idea/
/build/
/app/build/
```

---

### `README.md`

```markdown
# Android Video Editor (Java + FFmpeg)

Simple video editor Android app in Java.  
Trim videos and export in **720p** or **1080p** using FFmpeg.

## Features

- Select video from gallery  
- Trim using start & end time  
- Export in 720p or 1080p  
- Simple UI  

## Tech Stack

- Android (Java)  
- Mobile‑FFmpeg (com.arthenica:mobile-ffmpeg-full:4.4.LTS)  

## Usage

1. Clone or Download this project  
2. Open in Android Studio  
3. Allow storage permissions  
4. Run on a real device (not emulator)  
5. Select a video → set trim times → choose resolution → export  

## FFmpeg Command Sample

```

-i input.mp4 -ss 00:00:05 -to 00:00:15 -vf scale=-1:720 -c:v libx264 -preset ultrafast -c:a copy output_720p.mp4

```

Change `scale=-1:1080` for 1080p.

## License

MIT

## Author

(Your Name)
```

---

### `build.gradle` (root)

```gradle
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:7.0.0'
    }
}
allprojects {
    repositories {
        google()
        mavenCentral()
    }
}
```

---

### `app/build.gradle`

```gradle
plugins {
    id 'com.android.application'
}

android {
    compileSdk 33

    defaultConfig {
        applicationId "com.example.androidvideoeditor"
        minSdk 21
        targetSdk 33
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }
}

dependencies {
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    implementation 'com.arthenica:mobile-ffmpeg-full:4.4.LTS'
}
```

---

### `AndroidManifest.xml`

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.androidvideoeditor">

    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>

    <application
        android:allowBackup="true"
        android:label="AndroidVideoEditor"
        android:theme="@style/Theme.AppCompat.Light.NoActionBar">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
```

---

### `res/values/strings.xml`

```xml
<resources>
    <string name="app_name">AndroidVideoEditor</string>
</resources>
```

---

### `res/layout/activity_main.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:padding="16dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Button
        android:id="@+id/btn_pick_video"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Select Video" />

    <EditText
        android:id="@+id/start_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Start time (e.g. 00:00:05)" />

    <EditText
        android:id="@+id/end_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="End time (e.g. 00:00:15)" />

    <RadioGroup
        android:id="@+id/resolution_group"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <RadioButton
            android:id="@+id/rb_720"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="720p" />

        <RadioButton
            android:id="@+id/rb_1080"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="1080p" />
    </RadioGroup>

    <Button
        android:id="@+id/btn_export"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Export Video" />

</LinearLayout>
```

---

### `java/com/example/androidvideoeditor/MainActivity.java`

```java
package com.example.androidvideoeditor;

import android.Manifest;
import android.app.Activity;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.os.Environment;
import android.provider.MediaStore;
import android.widget.Button;
import android.widget.EditText;
import android.widget.RadioGroup;
import android.widget.Toast;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

public class MainActivity extends AppCompatActivity {

    private static final int PICK_VIDEO = 100;
    private static final int REQUEST_PERMISSIONS = 200;

    private String inputPath;
    EditText startTime, endTime;
    RadioGroup resolutionGroup;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        startTime = findViewById(R.id.start_time);
        endTime = findViewById(R.id.end_time);
        resolutionGroup = findViewById(R.id.resolution_group);
        Button pickVideo = findViewById(R.id.btn_pick_video);
        Button exportBtn = findViewById(R.id.btn_export);

        pickVideo.setOnClickListener(v -> {
            if (checkAndRequestPermissions()) {
                pickVideoFromGallery();
            }
        });

        exportBtn.setOnClickListener(v -> {
            if (inputPath == null) {
                Toast.makeText(this, "Please select a video first", Toast.LENGTH_SHORT).show();
                return;
            }
            String start = startTime.getText().toString().trim();
            String end = endTime.getText().toString().trim();
            if (start.isEmpty() || end.isEmpty()) {
                Toast.makeText(this, "Set start and end time", Toast.LENGTH_SHORT).show();
                return;
            }

            int selectedId = resolutionGroup.getCheckedRadioButtonId();
            String resolution = (selectedId == R.id.rb_720) ? "720" : "1080";

            String outputPath = Environment.getExternalStorageDirectory().getPath()
                    + "/Download/output_" + resolution + "p.mp4";

            VideoUtils.exportHD(inputPath, outputPath, start, end, resolution, MainActivity.this);
        });
    }

    private boolean checkAndRequestPermissions() {
        String[] permissions = {
            Manifest.permission.READ_EXTERNAL_STORAGE,
            Manifest.permission.WRITE_EXTERNAL_STORAGE
        };
        boolean allGranted = true;
        for (String perm : permissions) {
            if (ContextCompat.checkSelfPermission(this, perm) != PackageManager.PERMISSION_GRANTED) {
                allGranted = false;
                break;
            }
        }
        if (!allGranted) {
            ActivityCompat.requestPermissions(this, permissions, REQUEST_PERMISSIONS);
            return false;
        }
        return true;
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        if (requestCode == REQUEST_PERMISSIONS) {
            boolean grantedAll = true;
            for (int res : grantResults) {
                if (res != PackageManager.PERMISSION_GRANTED) {
                    grantedAll = false;
                    break;
                }
            }
            if (!grantedAll) {
                Toast.makeText(this, "Permissions are required", Toast.LENGTH_SHORT).show();
            }
        }
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    }

    private void pickVideoFromGallery() {
        Intent intent = new Intent(Intent.ACTION_PICK, MediaStore.Video.Media.EXTERNAL_CONTENT_URI);
        startActivityForResult(intent, PICK_VIDEO);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == PICK_VIDEO && resultCode == RESULT_OK && data != null) {
            Uri selectedUri = data.getData();
            inputPath = VideoUtils.getPath(this, selectedUri);
            Toast.makeText(this, "Video Selected: " + inputPath, Toast.LENGTH_SHORT).show();
        }
    }
}
```

---

### `java/com/example/androidvideoeditor/VideoUtils.java`

```java
package com.example.androidvideoeditor;

import android.app.Activity;
import android.content.Context;
import android.database.Cursor;
import android.net.Uri;
import android.provider.MediaStore;
import android.util.Log;
import android.widget.Toast;

import com.arthenica.mobileffmpeg.FFmpeg;
import com.arthenica.mobileffmpeg.Config;

public class VideoUtils {

    public static void exportHD(String inputPath, String outputPath,
                                String start, String end, String resolution,
                                Context context) {
        String scale = resolution.equals("720") ? "scale=-1:720" : "scale=-1:1080";

        String cmd = "-i \"" + inputPath + "\" -ss " + start + " -to " + end
                + " -vf " + scale
                + " -c:v libx264 -preset ultrafast -c:a copy \"" + outputPath + "\"";

        Log.d("VideoUtils", "CMD = " + cmd);

        FFmpeg.executeAsync(cmd, (executionId, returnCode) -> {
            if (returnCode == Config.RETURN_CODE_SUCCESS) {
                ((Activity) context).runOnUiThread(() ->
                    Toast.makeText(context, "Export complete: " + outputPath, Toast.LENGTH_LONG).show());
            } else {
                Log.e("VideoUtils", "Export failed, rc=" + returnCode);
                ((Activity) context).runOnUiThread(() ->
                    Toast.makeText(context, "Export failed!", Toast.LENGTH_LONG).show());
            }
        });
    }

    public static String getPath(Context context, Uri uri) {
        String[] projection = { MediaStore.Video.Media.DATA };
        Cursor cursor = context.getContentResolver().query(uri, projection, null, null, null);
        if (cursor != null) {
            int index = cursor.getColumnIndexOrThrow(MediaStore.Video.Media.DATA);
            cursor.moveToFirst();
            String path = cursor.getString(index);
            cursor.close();
            return path;
        }
        return null;
    }
}
```

---
