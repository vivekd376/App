# App
application 
print("My name is Dubey")

import java.io.File;
import java.io.FileInputStream;
import java.nio.charset.Charset;
import java.nio.file.Files;
import java.security.KeyFactory;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.SecureRandom;
import java.security.Security;
import java.security.cert.CertificateFactory;
import java.security.cert.X509Certificate;
import java.security.interfaces.RSAPrivateKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.text.SimpleDateFormat;
import java.util.Arrays;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;

import org.apache.commons.codec.binary.Base64;
import org.bouncycastle.jce.provider.BouncyCastleProvider;
import org.json.simple.JSONArray;
import org.json.simple.JSONObject;
import org.json.simple.parser.JSONParser;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.web.client.RestTemplate;

import sun.misc.BASE64Decoder;
import sun.security.provider.PolicyParser.ParsingException;

public class SimulatorNEFTGOK {
	SimpleDateFormat sd = new SimpleDateFormat("ddMMyyyyhhmmss");
	static long startTime = 0L;
	static long endTime = 0L;
	static JSONObject REQUEST = new JSONObject();
	static HttpHeaders httpHeader = new HttpHeaders();
	///ab
	//public static String IP_PORT="10.176.6.133:8501"; //SIT_SSO=8504/8510 
	public static String IP_PORT="10.176.6.136:8504"; //SIT_SSO=8504/8510
	public static String[] operations = {
			"pcmsNEFTLoad|CARDVALIDATE"/*0*/,"pcmsNEFTLoad|CARDLOAD"/*1*/
	};
	public static String request=operations[1];//CHANGE API NAME HERE
	public static String REQUEST_ID="NFT0601202515733008";//NFT1810201315310001
	public static String REQUEST_CODE_VALIDATE="CARDVALIDATION"; 
	public static String REQUEST_CODE_LOAD="CARDLOAD";
	
	//for Card Validate
	public static String SHORT_VAN="SBIGOK";
	public static String NEFTREFNO = "S181221512835595"; //Always New/unique P181220230035
	public static String IFSC_CODE = "CNRB0000987";//"CNRB0000987"; // CNRB0000987
	public static String TXNDATE="20122027";
	public static String AMOUNT_VALIDATE = "2999";
	public static String DDO_NO = "K2GOK12345";// PP:K2GOK12345 //  K2GOK11458 ----K2GOK072309 will be available in CMS_PROD_CARD_PAN  = CRP_DISP_NAM/"K2GOK072309"   = Proxy not required/12345678958//K2GOK654321//K2GOK072309/K2GOK072308//K2GOK072309
	
	//for Card Load
	public static String CORP_NAME = "MAHARASHTRA INFORMATION TECH CORP L";
	public static String UTR = "S181221512835595";  //Same as NEFTREFNO 
	public static String PARENT_UTR = "RBIPTRN29ABCD";
	public static String SENDER_IFSC = "INB0000987";
	public static String SENDER_ACCNO = "971";
	public static String SENDER_NAME ="VKhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhbB";
	public static String RECEIVER_IFSC = "swpN0004266";
	public static String RECEIVER_ACCNO = "3784474589";
	public static String DEALER_CODE = "01acd";
	public static String RECEIVER_ADDRESS = "Ba1";
	public static String AMOUNT_LOAD = "2999";  //Amount 
	public static String MIS_DATE = "20122028";
	public static String BATCH_TIME = "250000";
	public static String TXN_DATE = "20122023";
	public static String TXN_REMIT_DATE = "20122029";
	public static String FULL_VAN = "MAHAON012342232322323J";
	
	public static void main(String[] args) {
		sendRequestNew();
	}
	public static void getRequestMessage(){
		if(request.equalsIgnoreCase("pcmsNEFTLoad|CARDVALIDATE")){
			REQUEST.put("REQUEST_ID", REQUEST_ID);
			REQUEST.put("REQUEST_CODE", REQUEST_CODE_VALIDATE);
			JSONObject json1=new JSONObject();
			JSONArray jsonArray = new JSONArray();
			json1.put("SHORT_VAN", SHORT_VAN);
			json1.put("NEFTREFNO", NEFTREFNO);
			json1.put("IFSC_CODE", IFSC_CODE);
			json1.put("TXNDATE", TXNDATE);
			json1.put("AMOUNT", AMOUNT_VALIDATE);
			json1.put("DDO_NO", DDO_NO);
			jsonArray.add(json1);
			REQUEST.put("REQUEST_DATA", jsonArray);
		}else if(request.equalsIgnoreCase("pcmsNEFTLoad|CARDLOAD")){
			REQUEST.put("REQUEST_ID", REQUEST_ID);
			REQUEST.put("REQUEST_CODE", REQUEST_CODE_LOAD);
			JSONObject json1=new JSONObject();
			JSONArray jsonArray = new JSONArray();
			json1.put("CORP_NAME",CORP_NAME);
			json1.put("UTR",UTR);
			json1.put("PARENT_UTR",PARENT_UTR);
			json1.put("SENDER_IFSC",SENDER_IFSC);
			json1.put("SENDER_ACCNO",SENDER_ACCNO);
			json1.put("SENDER_NAME",SENDER_NAME);
			json1.put("RECEIVER_IFSC",RECEIVER_IFSC);
			json1.put("RECEIVER_ACCNO",RECEIVER_ACCNO);
			json1.put("DEALER_CODE",DEALER_CODE);
			json1.put("RECEIVER_ADDRESS",RECEIVER_ADDRESS);
			json1.put("AMOUNT",AMOUNT_LOAD);
			json1.put("MIS_DATE",MIS_DATE);
			json1.put("BATCH_TIME",BATCH_TIME);
			json1.put("TXN_DATE",TXN_DATE);
			json1.put("TXN_REMIT_DATE",TXN_REMIT_DATE);
			json1.put("FULL_VAN",FULL_VAN);
			jsonArray.add(json1);
			REQUEST.put("REQUEST_DATA", jsonArray);
			
		}
		else {
			System.out.println("Invalid Operation!!!!");
		}
	}
	
	//////////////////############## ENCRYPTION STARTS ###############////////////////
	static String key = null;
	public static void sendRequestNew() {
		try 
		{
			//String randomNum=String.valueOf((int)(Math.random()*9000)+1000);
			//SimpleDateFormat sd = new SimpleDateFormat("ddMMyyhhmmss");
			//REQUEST_ID=CHANNEL_ID+sd.format(new Date())+randomNum;
			getRequestMessage();
			System.out.println("Clear Request is:"+REQUEST);
			String aesKey = dynamicKeyGeneration();
			//System.out.println("Aes key "+aesKey);
			String aesEncryptedPayload = encryptResponse(REQUEST.toJSONString(),aesKey.getBytes());
			
			/*System.out.println("aesEncryptedPayload "+aesEncryptedPayload);
			System.out.println("aeskey :|"+aesKey+"|");
			System.out.println("aesDecryptedPayload "+decryptAES1(aesEncryptedPayload,aesKey.getBytes()));*/

			
			
			
			//FileInputStream fin = new FileInputStream("D:\\Shankar\\INS_KEYS\\EIS_TEMP_CERT.cer");
			FileInputStream fin = new FileInputStream("F:/Akash_Shared/AkashBNnew/AKASH Simulator Backup/EIS_KEY/pcms_publicKey/EIS_TEMP_CERT.cer");
		//	FileInputStream fin = new FileInputStream("E:/NEF_GOK/pcms_publicKey/EIS_KEY/pcms_publicKey/EIS_TEMP_CERT.cer");
			CertificateFactory f = CertificateFactory.getInstance("X.509");
			X509Certificate certificate = (X509Certificate)f.generateCertificate(fin);
			PublicKey pubKey = certificate.getPublicKey();
			//System.out.println("RSA Encrypted AES key "+aesKey);
			String rsaPublicEncryptedAesKey = RSAEncryption(aesKey,pubKey);
			//System.out.println("RSA Encrypted AES key "+rsaPublicEncryptedAesKey);
			key = rsaPublicEncryptedAesKey;
			JSONObject obj = new JSONObject(); 
			obj.put("REQUEST", aesEncryptedPayload);
			//System.out.println("Encrypted Request is:"+aesEncryptedPayload);
			System.out.println("Encrypted Request is:"+obj);
			//System.out.println("Full request"+obj);
			RestTemplate rest = new RestTemplate();
			httpHeader.setContentType(MediaType.APPLICATION_JSON);
			httpHeader.add("AccessToken", rsaPublicEncryptedAesKey);
			
			System.out.println(" rsaPublicEncryptedAesKey "+rsaPublicEncryptedAesKey);
			HttpEntity<String> entity = new HttpEntity<>(obj.toString(), httpHeader);
			if(request.equals("pcmsNEFTLoad|CARDVALIDATE")|| request.equals("pcmsNEFTLoad|CARDLOAD")) {
				request="pcmsNEFTLoad";
			}
			System.out.println("URL:"+"http://"+IP_PORT+"/SBICMS/cmsServices/"+request);
			String resMessage = rest.postForObject("http://"+IP_PORT+"/SBICMS/cmsServices/"+request,entity,String.class);
			System.out.println("Encrypted Response is:"+resMessage);
			JSONObject jsonRequest = new JSONObject();
			JSONObject json = (JSONObject) new JSONParser().parse(resMessage);
			//System.out.println("JSON Response "+json);
			System.out.println("Response is"+getDecryptedReuqest(json.get("RESPONSE").toString()));
			//System.out.println("Response is:"+RSAAndPKCS5PaddingDecryption(json.get("RESPONSE").toString(),rsaPublicEncryptedAesKey));
			
		} 
		catch (Exception e) {
			e.printStackTrace();
			System.out.println("Error:"+e.getMessage()); 
		}
	}
	
	public static String getDecryptedReuqest(String response) {
		String clearResponse="";
		String encryptedPaylaod=null;
		String encryptedAES=null;
		String decryptedAES = null;
		try {  
			encryptedPaylaod =response;
			encryptedAES=key;
			File f1 = new File("F:/Akash_Shared/AkashBNnew/AKASH Simulator Backup/EIS_KEY/pcms_privateKey/EIS_TEMP_CERT_KEY.cer");
		//	File f1 = new File("E:/NEF_GOK/pcms_publicKey/EIS_KEY/pcms_privateKey/EIS_TEMP_CERT_KEY.cer");
			PrivateKey PCMS_PrivateKey = readPrivateKey(f1);
			decryptedAES=doDecrypt(encryptedAES, PCMS_PrivateKey);
			//System.out.println("ASE KEY :|"+decryptedAES+"|");
			clearResponse=decryptAES(encryptedPaylaod,decryptedAES.getBytes());
			//System.out.println("clearResponse:"+clearResponse);
		} catch(ParsingException e){
			System.out.println("Error In Decryption:"+e.getMessage());
		}catch (Exception e) {
			e.printStackTrace();
			System.out.println("Error In Decryption:"+e.getMessage());
		}
		return clearResponse;
	}
	
	public static String doDecrypt(String dataToBeDecrypted,PrivateKey PrivateKey) throws Exception{
		String decryptedData = null;
		Cipher dipher;
		try
		{ 
			//dipher = Cipher.getInstance("RSA/ECB/OAEPWithMD5AndMGF1Padding");
			dipher = Cipher.getInstance("RSA/ECB/OAEPwithSHA-256andMGF1padding");
			dipher.init(Cipher.DECRYPT_MODE, PrivateKey);
			/*BASE64Decoder decoder = new BASE64Decoder();
			byte[] encrypted = decoder.decodeBuffer(dataToBeDecrypted);*/ 
			byte[] encrypted = Base64.decodeBase64(dataToBeDecrypted); 
			byte[] decrypted = dipher.doFinal(encrypted); 
			decryptedData =  new String(decrypted,"UTF-8");  
		}
		catch (Exception e)
		{
			e.printStackTrace();
			throw new Exception(e.getMessage());
		}
		return decryptedData;
	}
	
	public static  String decryptAES(String encText, byte[] aesKey)
			throws Exception 
	{
		byte[] encryptedTextByte =Base64.decodeBase64(encText);
		byte[] iv = null;   
		iv = Arrays.copyOfRange(aesKey, 0, 16);
		IvParameterSpec ivspec = new IvParameterSpec(iv);
		SecretKeySpec skey = new SecretKeySpec(aesKey, "AES");
		//Cipher ci = Cipher.getInstance("AES/CBC/PKCS5Padding");
		Security.addProvider(new BouncyCastleProvider());
		Cipher ci = Cipher.getInstance("AES/GCM/nopadding");
		ci.init(Cipher.DECRYPT_MODE, skey, ivspec);
		byte[] decryptedByte = ci.doFinal(encryptedTextByte);
		String decryptedText = new String(decryptedByte);
		return decryptedText;
	}
	
	public static String dynamicKeyGeneration()
	{
		byte[] raw;
		String aesKeyB64="";
		try
		{
			KeyGenerator keyGen = KeyGenerator.getInstance("AES");
			SecureRandom rn = new SecureRandom();
			keyGen.init(256,rn);
			SecretKey secretKey = keyGen.generateKey();
			raw = secretKey.getEncoded();
			//writeUsingOutputStream(raw);
			//String aesKeyB64 = Base64.encodeBase64(raw);
			aesKeyB64= Base64.encodeBase64String(raw).substring(0, 32);
		}
		catch(Exception ex)
		{
			ex.printStackTrace();
		}
		return aesKeyB64;
	}
	
	public static String encryptResponse(String plainText,byte[] aesKey)
			throws Exception 
	{
		byte[] plainTextByte = plainText.getBytes();

		Security.addProvider(new BouncyCastleProvider());
		Cipher ci = Cipher.getInstance("AES/GCM/nopadding");
		//Cipher ci = Cipher.getInstance("AES/CBC/PKCS5Padding"); //Cipher Mode Operation
		byte[] iv = null;
		iv = Arrays.copyOfRange(aesKey, 0, 16);
		IvParameterSpec ivspec = new IvParameterSpec(iv);
		SecretKeySpec skey = new SecretKeySpec(aesKey,"AES");
		ci.init(Cipher.ENCRYPT_MODE, skey, ivspec);	
		byte[] encryptedByte = ci.doFinal(plainTextByte);
		String encryptedText=Base64.encodeBase64String(encryptedByte);
		return encryptedText;
	}
	
	private static String RSAEncryption(String dataToBeEncrypted,PublicKey publicKey) throws Exception {
		String rSAEncryption="";
        try {
        	//System.out.println("Clear AES KEY "+dataToBeEncrypted);
        	//System.out.println("AES KEY Bytes "+dataToBeEncrypted.getBytes());
        	//System.out.println("Base64 Encoded bytes "+Base64.encodeBase64String(dataToBeEncrypted.getBytes()));
        	//Cipher cipher = Cipher.getInstance("RSA/ECB/OAEPWithSHA-256AndMGF1Padding");
        	//Cipher cipher = Cipher.getInstance("RSA/ECB/OAEPWithMD5AndMGF1Padding");
        	Cipher cipher = Cipher.getInstance("RSA/ECB/OAEPwithSHA-256andMGF1padding");
			cipher.init(Cipher.ENCRYPT_MODE, publicKey);
			byte[] ciphertext = cipher.doFinal(dataToBeEncrypted.getBytes());
			rSAEncryption= Base64.encodeBase64String(ciphertext);
		} catch (Exception e) {
			e.printStackTrace();
		}
        return rSAEncryption;
    }
	
	public static  String decryptAES1(String encText, byte[] aesKey)
			throws Exception 
	{
		byte[] encryptedTextByte =Base64.decodeBase64(encText);
		byte[] iv = null;   
		iv = Arrays.copyOfRange(aesKey, 0, 16);
		IvParameterSpec ivspec = new IvParameterSpec(iv);
		SecretKeySpec skey = new SecretKeySpec(aesKey, "AES");
		//Cipher ci = Cipher.getInstance("AES/CBC/PKCS5Padding");
		Security.addProvider(new BouncyCastleProvider());
	    Cipher ci = Cipher.getInstance("AES/GCM/nopadding");
		ci.init(Cipher.DECRYPT_MODE, skey, ivspec);
		byte[] decryptedByte = ci.doFinal(encryptedTextByte);
		String decryptedText = new String(decryptedByte);
		return decryptedText;
	}
	
	public static RSAPrivateKey readPrivateKey(File file) throws Exception {
		String key = new String(Files.readAllBytes(file.toPath()), Charset.defaultCharset());

		String privateKeyPEM = key
				.replace("-----BEGIN PRIVATE KEY-----", "")
				.replaceAll(System.lineSeparator(), "")
				.replace("-----END PRIVATE KEY-----", "");

		byte[] encoded = Base64.decodeBase64(privateKeyPEM);

		KeyFactory keyFactory = KeyFactory.getInstance("RSA");
		PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(encoded);
		return (RSAPrivateKey) keyFactory.generatePrivate(keySpec);
	}
	
	/*private static String RSADecryption(String dataToBeEncrypted,PrivateKey privateKey) throws Exception {
		String rSDecryption="";
        try {
	    Cipher cipher = Cipher.getInstance("RSA/ECB/OAEPWithSHA-256AndMGF1Padding");
        cipher.init(Cipher.DECRYPT_MODE, privateKey);
        byte[] ciphertext = Base64.decodeBase64(dataToBeEncrypted);
        byte[] decryptedData = cipher.doFinal(ciphertext);
        rSDecryption= new String(decryptedData);
		} catch (Exception e) {
			e.printStackTrace();
		}
        return rSDecryption;
    }*/
	
	public static String decryptPKCS5Padding(String encText, byte[] aesKey)
		    throws Exception
		  {
		    byte[] encryptedTextByte = Base64.decodeBase64(encText);
		    byte[] iv = null;
		    iv = Arrays.copyOfRange(aesKey, 0, 16);
		    IvParameterSpec ivspec = new IvParameterSpec(iv);
		    SecretKeySpec skey = new SecretKeySpec(aesKey, "AES");
		    Security.addProvider(new BouncyCastleProvider());
		    Cipher ci = Cipher.getInstance("AES/GCM/nopadding");
		    
		    ci.init(2, skey, ivspec);
		    byte[] decryptedByte = ci.doFinal(encryptedTextByte);
		    String decryptedText = new String(decryptedByte);
		    return decryptedText;
		  }

}
