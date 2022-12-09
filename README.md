#Get Selected Pdf File Uri 


#Selecting the  Pdf File to get uri to Upload at  api . 



@Prerequistic



1> In manifest file 

a>> 

            <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
            <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

b>> In <application >  tag 

        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.provider"
            android:exported="false"
            android:grantUriPermissions="true"
            tools:replace="android:authorities">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths"
                tools:replace="android:resource" />
        </provider>

c>> In res -> xml  . Create  a file_paths.xml file  and paste following code into it . 
   
           <?xml version="1.0" encoding="utf-8"?>
              <paths>
                    <external-path name="my_images" path="Android/data/<write_your_package_here>" />
                        <external-path
                            name="external_files"  
                            path="." />
          
                    <external-path name="external_files" path="."/>
            
                    <external-path name="external" path="." />
            
                    <cache-path name="cache" path="." />
          
                    <external-cache-path name="external_cache" path="." />
            
                    <files-path name="files" path="." />
        
                </paths>

2>   In   Project-level  Build.gradle file -->  Inside allprojects { }

       repositories {
        google()
        jcenter()
        maven { url "https://jitpack.io" }
        flatDir {
            dirs 'libs'
        }
    }

                                                
Steps ===>> 


1> IN select button onClick Function 

                Intent pdfIntent = new Intent();
                pdfIntent.setType("application/pdf");
                pdfIntent.addCategory(Intent.CATEGORY_OPENABLE);
                pdfIntent.setAction(Intent.ACTION_GET_CONTENT);
                startActivityForResult(pdfIntent, 12);              

2>  Override onActivityResult outside  the   outside  onCreate() function 

                if (requestCode == 12 && resultCode == RESULT_OK) {
      
            // Get the Uri of the selected file
            Uri uri = data.getData();
            String uriString = uri.toString();
            File myFile = new File(uriString);
            String displayName = null;
        
            if (uriString.startsWith("content://")) {
                Cursor cursor = null;
                try {
                    cursor = getContentResolver().query(uri, null, null, null, null);
                    if (cursor != null && cursor.moveToFirst()) {
                        displayName = cursor.getString(cursor.getColumnIndex(OpenableColumns.DISPLAY_NAME));
                    }
                } finally {
                    cursor.close();
                }
            } else if (uriString.startsWith("file://")) {
                displayName = myFile.getName();
            }
            Log.d("aksjdhasd",myFile.getPath());
    //        Log.d("aksjdhasd",new getPdfFile().getPdfFile(this,uri).getPath());

            
            etFile.setText(displayName.toString());
        
            uploadPdf(new GetFileFromUriUsingBufferReader().getFile(this,uri));
      
        }

3>  Create GetFileFromUriUsingBufferReader Kotlin Class and write following code into that class 
                  
          public  class GetFileFromUriUsingBufferReader {
            
        
            fun getFile(mContext: Activity?, documentUri: Uri): File {
                val inputStream = mContext?.contentResolver?.openInputStream(documentUri)
                var file = File("")
                inputStream.use { input ->
                    file =
                        File(mContext?.cacheDir, System.currentTimeMillis().toString() + ".pdf")
                    FileOutputStream(file).use { output ->
                        val buffer =
                            ByteArray(4 * 1024) // or other buffer size
                        var read: Int = -1
                        while (input?.read(buffer).also {
                                if (it != null) {
                                    read = it
                                }
                            } != -1) {
                            output.write(buffer, 0, read)
                        }
                        output.flush()
                    }
                  }
                  return file
                }
           }

4>>    Now return to the Activity &&   Create  uploadPdf void funtion outside the onActivityResult() 

    private void uploadPdf(File path) {
     Log.d(TAG, "uploadPdf: "+ path);
        Toast.makeText(this, ""+ path, Toast.LENGTH_SHORT).show();
              
        //  Now Call the api  using retrofit  and pass the value of path into the Api as following example 
        
        RestClient.getInst().upload_a_pdf(path).enqueue(new HttpCallback<Upload_astrologer_imageBean>() {
            @Override
            public void onSuccess(Call<Upload_astrologer_imageBean> call, Response<Upload_astrologer_imageBean> response) {
    
                if (response.body().getStatus()) {
                    Log.d(TAG, "onSuccess: "+response.body().getFile_img());
                                        pdfFile = response.body().getFile_img();
                    dialogLoader.hideProgressDialog();
    
                } else {
                    Log.e("Data11", "abc " + file2.getAbsolutePath());
                    // dialogLoader.hideProgressDialog();
                }
      
            }
    
            @Override
            public void onError(Call<Upload_astrologer_imageBean> call, Throwable t) {
                Log.e("Data11", "abc " + t.getMessage());
                dialogLoader.hideProgressDialog();
            }
        });
        
