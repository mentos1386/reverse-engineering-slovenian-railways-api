# Reverse engineering Slovenian Railways API


### Endpoints

__Base URL:__ `https://eshop.sz.si/SZMobile/`

```
/* renamed from: n2 */
/* loaded from: classes.dex */
public interface SZEshopService {
    @POST("authenticate_enc/")
    ta1<C2980uv> authenticateDevice(@Body SZEshopEncryptedRequestBody sZEshopEncryptedRequestBody);

    @POST("GetCard_enc/")
    ta1<C2980uv> getCard(@Body SZEshopEncryptedRequestBody sZEshopEncryptedRequestBody);

    @POST("GetCatalogs/")
    ta1<C2980uv> getCatalogs(@Body SZEshopEncryptedRequestBody sZEshopEncryptedRequestBody);

    @POST("get_messages_enc/")
    ta1<C2980uv> getMessages(@Body SZEshopEncryptedRequestBody sZEshopEncryptedRequestBody);

    @POST("GetOvire_enc/")
    ta1<C2980uv> getObstacles(@Body SZEshopEncryptedRequestBody sZEshopEncryptedRequestBody);

    @Headers({"CONNECT_TIMEOUT:60000", "READ_TIMEOUT:60000", "WRITE_TIMEOUT:60000"})
    @GET("https://eshop.sz.si/Purchase/DownloadReceiptAsPdf")
    Call<ResponseBody> getPdfTicketFile(@Query("receiptNumber") String str);

    @POST("GetProducts_enc/")
    ta1<C2980uv> getProducts(@Body SZEshopEncryptedRequestBody sZEshopEncryptedRequestBody);

    @POST("GetReceipt_enc/")
    ta1<C2980uv> getReceipt(@Body SZEshopEncryptedRequestBody sZEshopEncryptedRequestBody);

    @POST("GetRoutesSz_enc/")
    ta1<C2980uv> getRoutes(@Body SZEshopEncryptedRequestBody sZEshopEncryptedRequestBody);

    @POST("GetStations_enc/")
    ta1<C2980uv> getStations(@Body SZEshopEncryptedRequestBody sZEshopEncryptedRequestBody);

    @POST("GetSzCardProducts_enc/")
    ta1<C2980uv> getSzCardProducts(@Body SZEshopEncryptedRequestBody sZEshopEncryptedRequestBody);

    @POST("SetReceipt_enc/")
    ta1<C2980uv> setReceipt(@Body SZEshopEncryptedRequestBody sZEshopEncryptedRequestBody);

    @POST("SetReceiptSz_enc/")
    ta1<C2980uv> setReceiptNew(@Body SZEshopEncryptedRequestBody sZEshopEncryptedRequestBody);

    @POST("update_message_enc/")
    ta1<C2980uv> updateMessage(@Body SZEshopEncryptedRequestBody sZEshopEncryptedRequestBody);
}
```

#### `SZEshopEncryptedRequestBody`

```
{
    "req": "encrypted contents",
    "auth_token": "plain text token, looks like uuid"
}
```

### How to get token?

```
    @Override // p000.InterfaceSZEshopClient
    public void authenticateDevice(SZResponse<SZResponseAuthnticateDeviceResult> sZResponse, String serial) {
        CryptoUtil.configurePublicAndPrivateKeys(this.sharedPreferences);
        SZAuthenticateDeviceRequest sZAuthenticateDeviceRequest = new SZAuthenticateDeviceRequest();
        sZAuthenticateDeviceRequest.setDevice_serial(serial); // can be anything?
        sZAuthenticateDeviceRequest.setDevice_description(C1954lk.f7875a); // "Android " + Build.VERSION.SDK_INT + "/" + Build.MANUFACTURER + " " + Build.MODEL
        sZAuthenticateDeviceRequest.setApp_version("1.0.26");
        sZAuthenticateDeviceRequest.setPlatform_id(C2407pb.f9245a.intValue()); // 1
        sZAuthenticateDeviceRequest.setKey(CryptoUtil.encryptPartAES());
        sZAuthenticateDeviceRequest.setPublic_key(CryptoUtil.getShortPublicKey());
        getData(this.service.authenticateDevice(getRequest(sZAuthenticateDeviceRequest)), sZResponse, SZResponseAuthnticateDeviceResult.class, true);
    }
```

### How to perform encryption and decryption?

```
    /* renamed from: a */
    public static byte[] AESDecryptResponse(byte[] response, Context context) {
        if (deviceAuthenticationKey != null) {
            try {
                Cipher cipher = Cipher.getInstance("AES/CBC/PKCS7Padding");
                cipher.init(2, new SecretKeySpec(deviceAuthenticationKey, "AES"), new IvParameterSpec(m9812g()));
                return cipher.doFinal(response);
            } catch (Exception e) {
                String str = className;
                Logger.verbose(str, "AESDecryptResponse: " + e.getMessage());
                e.printStackTrace();
                return null;
            }
        }
        return m9802p(m9799v(response, context));
    }

    /* renamed from: b */
    public static String AESEncryptRequest(String request, Context context) {
        if (deviceAuthenticationKey != null) {
            try {
                Cipher cipher = Cipher.getInstance("AES/CBC/PKCS7Padding");
                cipher.init(1, new SecretKeySpec(deviceAuthenticationKey, "AES"), new IvParameterSpec(m9812g()));
                return Base64.encodeToString(cipher.doFinal(request.getBytes()), 2);
            } catch (Exception e) {
                String str = className;
                Logger.verbose(str, "AESEncryptRequest: " + e.getMessage());
                e.printStackTrace();
                return null;
            }
        }
        return Base64.encodeToString(m9798w(m9814e(request.getBytes()), context), 2);
    }
```
