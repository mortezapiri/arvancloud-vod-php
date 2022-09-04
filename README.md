

#### sample upload video to arvancloud vod

	use Illuminate\Http\Request;
    use GuzzleHttp\Psr7\Request as GuzzleRequest;

	Route::post('upload', function (Request $request) {
        $channel = 'channel_id_here';
        $api_key = 'Apikey_here';
        $target = "https://napi.arvancloud.com/vod/2.0/channels/{$channel}/files";
        $file_name = base64_encode($request->file->getClientOriginalName());
        $file_ext = base64_encode('video/' . $request->file->getClientOriginalExtension());
    
        $client = new \GuzzleHttp\Client();
        $headers = [
            'Authorization' => $api_key,
            'tus-resumable' => '1.0.0',
            'upload-length' => $request->file->getSize(),
            'upload-metadata' => "filename {$file_name},filetype {$file_ext}",
        ];
    
        $new_upload_file = new GuzzleRequest('POST', $target, $headers);
        $upload_file_result = $client->sendAsync($new_upload_file)->wait();
    
        if ($upload_file_result->getStatusCode() == 201) {
            $location = $upload_file_result->getHeaders()['Location'][0];
            $headers = [
                'Authorization' => $api_key,
                'tus-resumable' => '1.0.0',
                'upload-offset' => '0',
                'Content-Type' => 'application/offset+octet-stream'
            ];
            $apply_request = new GuzzleRequest('PATCH', $location, $headers, fopen($request->file->getRealPath(), "r"));
            $client->sendAsync($apply_request)->wait();
        }
    });
