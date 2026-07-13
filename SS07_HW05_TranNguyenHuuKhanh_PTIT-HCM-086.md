# SS05 - HOMEWORK 05

# BÀI 5: Thiết kế cổng giải mã cấu hình bảo mật

## 1. Ý đồ thiết kế

Module `SecurityDecryptor` chịu trách nhiệm giải mã mật khẩu cơ sở dữ liệu đã được mã hóa trước khi ứng dụng sử dụng để kết nối database.

Quy trình thiết kế gồm:

1. Nhận ciphertext dạng Base64 và secret key từ cấu hình hoặc biến môi trường.
2. Kiểm tra đầu vào trước khi gọi API mã hóa.
3. Chuyển secret key thành khóa AES hợp lệ.
4. Giải mã bằng `javax.crypto`.
5. Chuyển các lỗi kỹ thuật thành `DecryptionException`.
6. Ghi log an toàn, không ghi ciphertext, plaintext, secret key hoặc dữ liệu nhạy cảm.
7. Viết JUnit 5 để kiểm tra:
   - Giải mã thành công.
   - Ciphertext null/rỗng.
   - Secret key null/rỗng.
   - Key sai độ dài.
   - Sai key.
   - Ciphertext không phải Base64.
   - Dữ liệu mã hóa bị hỏng.

Để minh họa đơn giản và có thể chạy bằng thư viện mặc định Java 17, ví dụ sử dụng `AES/GCM/NoPadding`. So với ECB, GCM cung cấp cả tính bí mật và kiểm tra toàn vẹn. Dữ liệu mã hóa được quy ước gồm 12 byte IV ở đầu, tiếp theo là ciphertext và authentication tag, sau đó toàn bộ được Base64 encode.

---

# 2. Prompt gửi AI

```text
Bạn hãy đóng vai trò là một Senior Java Security Engineer.

[Bối cảnh]
Tôi đang xây dựng module SecurityDecryptor cho ứng dụng tài chính.
Module có nhiệm vụ giải mã mật khẩu database được truyền dưới dạng
chuỗi cấu hình Base64 hoặc được lấy từ biến môi trường.

[Mục tiêu]
Hãy sinh mã nguồn Java 17 hoàn chỉnh cho lớp SecurityDecryptor có phương thức:

String decrypt(String encryptedPassword, String secretKey)

[Ràng buộc kỹ thuật]
1. Chỉ sử dụng thư viện mặc định của Java 17, đặc biệt là javax.crypto.
2. Sử dụng AES/GCM/NoPadding thay vì AES/ECB.
3. encryptedPassword là chuỗi Base64 gồm:
   - 12 byte IV ở đầu.
   - Phần còn lại là ciphertext và authentication tag.
4. secretKey là chuỗi UTF-8 có độ dài hợp lệ cho AES:
   16, 24 hoặc 32 byte.
5. Dùng SecretKeySpec, Cipher và GCMParameterSpec.
6. Tạo ngoại lệ runtime tùy chỉnh DecryptionException.
7. Không trả về null khi giải mã thất bại.
8. Không nuốt ngoại lệ.
9. Sử dụng Lombok @Slf4j hoặc SLF4J để logging.
10. Tuyệt đối không log:
    - Mật khẩu sau giải mã.
    - Chuỗi encryptedPassword.
    - secretKey.
    - Nội dung nhạy cảm trong exception message.

[Kịch bản What-if]
Hãy phân tích và xử lý rõ các trường hợp sau:
1. encryptedPassword là null, rỗng hoặc chỉ có khoảng trắng.
2. secretKey là null, rỗng hoặc chỉ có khoảng trắng.
3. secretKey không có độ dài 16, 24 hoặc 32 byte.
4. encryptedPassword không phải Base64 hợp lệ.
5. Dữ liệu Base64 ngắn hơn độ dài IV.
6. Dùng sai secretKey làm phát sinh AEADBadTagException hoặc
   BadPaddingException.
7. Khóa không hợp lệ làm phát sinh InvalidKeyException.
8. Ciphertext bị thay đổi hoặc hỏng.
9. Ghi log cảnh báo an toàn mà không làm rò rỉ dữ liệu mật khẩu.

[Kiểm thử]
Sinh thêm class JUnit 5 test hoàn chỉnh.
Trong test được phép tạo helper encrypt chỉ để chuẩn bị dữ liệu kiểm thử.
Phải có test cho:
- Giải mã thành công.
- encryptedPassword null, empty và blank.
- secretKey null, empty và blank.
- key sai độ dài.
- Base64 không hợp lệ.
- payload quá ngắn.
- sai key.
- ciphertext bị sửa.
- xác minh các lỗi được chuyển thành DecryptionException.

[Đầu ra]
1. Giải thích ngắn gọn thiết kế.
2. SecurityDecryptor.java.
3. DecryptionException.java.
4. SecurityDecryptorTest.java.
5. Dependency Maven cần thiết cho Lombok, SLF4J và JUnit 5.
6. Tất cả code phải nằm trong khối markdown và có thể biên dịch.
```

---

# 3. Minh chứng log chat dạng Markdown

Phần dưới đây là nội dung tương tác dạng text/markdown minh họa quá trình chạy prompt.

## User

```text
Bạn hãy đóng vai trò là một Senior Java Security Engineer.

Tôi cần lớp SecurityDecryptor Java 17 có phương thức
decrypt(String encryptedPassword, String secretKey), sử dụng
AES/GCM/NoPadding và javax.crypto.

Ciphertext được Base64 encode, gồm 12 byte IV ở đầu và phần còn lại
là ciphertext kèm authentication tag.

Hãy xử lý các trường hợp encryptedPassword rỗng, secretKey null,
key sai độ dài, Base64 sai, sai secret key, BadPaddingException,
InvalidKeyException và ciphertext bị hỏng.

Mọi lỗi phải được chuyển thành DecryptionException. Chỉ ghi log an toàn,
không log ciphertext, plaintext hoặc secret key.

Hãy sinh code hoàn chỉnh và JUnit 5 test đi kèm.
```

## Assistant

```text
Thiết kế sử dụng AES/GCM/NoPadding. Phương thức decrypt kiểm tra đầu vào,
giải mã Base64, tách IV 12 byte, tạo SecretKeySpec và gọi Cipher#doFinal.
Các lỗi do dữ liệu đầu vào, sai khóa hoặc payload bị sửa được chuyển thành
DecryptionException. Log chỉ mô tả loại thao tác thất bại và không chứa
ciphertext, plaintext hoặc secret key.

Dưới đây là mã nguồn hoàn chỉnh.
```

## Assistant - `DecryptionException.java`

```java
public class DecryptionException extends RuntimeException {

    public DecryptionException(String message) {
        super(message);
    }

    public DecryptionException(
            String message,
            Throwable cause
    ) {
        super(message, cause);
    }
}
```

## Assistant - `SecurityDecryptor.java`

```java
import lombok.extern.slf4j.Slf4j;

import javax.crypto.AEADBadTagException;
import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.spec.GCMParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.nio.ByteBuffer;
import java.nio.charset.StandardCharsets;
import java.security.InvalidAlgorithmParameterException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.util.Base64;

@Slf4j
public final class SecurityDecryptor {

    private static final String TRANSFORMATION = "AES/GCM/NoPadding";
    private static final String KEY_ALGORITHM = "AES";
    private static final int IV_LENGTH_BYTES = 12;
    private static final int GCM_TAG_LENGTH_BITS = 128;

    public String decrypt(
            String encryptedPassword,
            String secretKey
    ) {
        validateEncryptedPassword(encryptedPassword);
        byte[] keyBytes = validateAndGetKeyBytes(secretKey);

        byte[] payload;
        try {
            payload = Base64.getDecoder().decode(encryptedPassword);
        } catch (IllegalArgumentException exception) {
            log.warn(
                    "Database credential decryption failed: "
                            + "invalid encoded payload."
            );
            throw new DecryptionException(
                    "Encrypted password has an invalid format.",
                    exception
            );
        }

        if (payload.length <= IV_LENGTH_BYTES) {
            log.warn(
                    "Database credential decryption failed: "
                            + "payload structure is invalid."
            );
            throw new DecryptionException(
                    "Encrypted password payload is invalid."
            );
        }

        byte[] initializationVector =
                new byte[IV_LENGTH_BYTES];
        byte[] cipherText =
                new byte[payload.length - IV_LENGTH_BYTES];

        ByteBuffer.wrap(payload)
                .get(initializationVector)
                .get(cipherText);

        try {
            Cipher cipher = Cipher.getInstance(TRANSFORMATION);

            SecretKeySpec keySpec =
                    new SecretKeySpec(keyBytes, KEY_ALGORITHM);

            GCMParameterSpec parameterSpec =
                    new GCMParameterSpec(
                            GCM_TAG_LENGTH_BITS,
                            initializationVector
                    );

            cipher.init(
                    Cipher.DECRYPT_MODE,
                    keySpec,
                    parameterSpec
            );

            byte[] plainText = cipher.doFinal(cipherText);
            return new String(plainText, StandardCharsets.UTF_8);
        } catch (AEADBadTagException exception) {
            log.warn(
                    "Database credential decryption failed: "
                            + "authentication check failed."
            );
            throw new DecryptionException(
                    "Cannot decrypt database credential.",
                    exception
            );
        } catch (BadPaddingException exception) {
            log.warn(
                    "Database credential decryption failed: "
                            + "invalid padding or key."
            );
            throw new DecryptionException(
                    "Cannot decrypt database credential.",
                    exception
            );
        } catch (InvalidKeyException exception) {
            log.error(
                    "Database credential decryption failed: "
                            + "invalid AES key."
            );
            throw new DecryptionException(
                    "The configured decryption key is invalid.",
                    exception
            );
        } catch (InvalidAlgorithmParameterException exception) {
            log.error(
                    "Database credential decryption failed: "
                            + "invalid cipher parameters."
            );
            throw new DecryptionException(
                    "The encrypted password parameters are invalid.",
                    exception
            );
        } catch (NoSuchAlgorithmException |
                 NoSuchPaddingException exception) {
            log.error(
                    "Database credential decryption failed: "
                            + "required cryptographic algorithm "
                            + "is unavailable."
            );
            throw new DecryptionException(
                    "Required cryptographic algorithm is unavailable.",
                    exception
            );
        } catch (IllegalBlockSizeException exception) {
            log.warn(
                    "Database credential decryption failed: "
                            + "invalid encrypted block."
            );
            throw new DecryptionException(
                    "Encrypted password payload is invalid.",
                    exception
            );
        } finally {
            java.util.Arrays.fill(keyBytes, (byte) 0);
        }
    }

    private void validateEncryptedPassword(
            String encryptedPassword
    ) {
        if (encryptedPassword == null
                || encryptedPassword.isBlank()) {
            throw new DecryptionException(
                    "Encrypted password must not be null or blank."
            );
        }
    }

    private byte[] validateAndGetKeyBytes(String secretKey) {
        if (secretKey == null || secretKey.isBlank()) {
            throw new DecryptionException(
                    "Secret key must not be null or blank."
            );
        }

        byte[] keyBytes =
                secretKey.getBytes(StandardCharsets.UTF_8);

        if (keyBytes.length != 16
                && keyBytes.length != 24
                && keyBytes.length != 32) {
            java.util.Arrays.fill(keyBytes, (byte) 0);
            throw new DecryptionException(
                    "Secret key must contain 16, 24, or 32 UTF-8 bytes."
            );
        }

        return keyBytes;
    }
}
```

## Assistant - `SecurityDecryptorTest.java`

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.NullAndEmptySource;
import org.junit.jupiter.params.provider.ValueSource;

import javax.crypto.Cipher;
import javax.crypto.spec.GCMParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.nio.ByteBuffer;
import java.nio.charset.StandardCharsets;
import java.security.SecureRandom;
import java.util.Base64;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

class SecurityDecryptorTest {

    private static final String VALID_KEY =
            "1234567890abcdef";
    private static final String WRONG_KEY =
            "abcdef1234567890";

    private SecurityDecryptor decryptor;

    @BeforeEach
    void setUp() {
        decryptor = new SecurityDecryptor();
    }

    @Test
    @DisplayName("Should decrypt a valid encrypted password")
    void shouldDecryptValidPassword() throws Exception {
        String originalPassword = "Db@Password123";
        String encryptedPassword =
                encrypt(originalPassword, VALID_KEY);

        String decryptedPassword =
                decryptor.decrypt(encryptedPassword, VALID_KEY);

        assertEquals(originalPassword, decryptedPassword);
    }

    @ParameterizedTest
    @NullAndEmptySource
    @ValueSource(strings = {" ", "   "})
    @DisplayName("Should reject null or blank encrypted password")
    void shouldRejectNullOrBlankEncryptedPassword(
            String encryptedPassword
    ) {
        assertThrows(
                DecryptionException.class,
                () -> decryptor.decrypt(
                        encryptedPassword,
                        VALID_KEY
                )
        );
    }

    @ParameterizedTest
    @NullAndEmptySource
    @ValueSource(strings = {" ", "   "})
    @DisplayName("Should reject null or blank secret key")
    void shouldRejectNullOrBlankSecretKey(String secretKey) {
        assertThrows(
                DecryptionException.class,
                () -> decryptor.decrypt(
                        "dGVzdA==",
                        secretKey
                )
        );
    }

    @ParameterizedTest
    @ValueSource(strings = {
            "short-key",
            "123456789012345",
            "12345678901234567",
            "1234567890123456789012345678901",
            "123456789012345678901234567890123"
    })
    @DisplayName("Should reject secret key with invalid AES length")
    void shouldRejectInvalidKeyLength(String secretKey) {
        assertThrows(
                DecryptionException.class,
                () -> decryptor.decrypt(
                        "dGVzdA==",
                        secretKey
                )
        );
    }

    @Test
    @DisplayName("Should reject invalid Base64")
    void shouldRejectInvalidBase64() {
        assertThrows(
                DecryptionException.class,
                () -> decryptor.decrypt(
                        "not-valid-base64***",
                        VALID_KEY
                )
        );
    }

    @Test
    @DisplayName("Should reject payload shorter than IV")
    void shouldRejectShortPayload() {
        String shortPayload = Base64.getEncoder()
                .encodeToString(new byte[5]);

        assertThrows(
                DecryptionException.class,
                () -> decryptor.decrypt(
                        shortPayload,
                        VALID_KEY
                )
        );
    }

    @Test
    @DisplayName("Should reject an incorrect secret key")
    void shouldRejectIncorrectSecretKey() throws Exception {
        String encryptedPassword =
                encrypt("Db@Password123", VALID_KEY);

        assertThrows(
                DecryptionException.class,
                () -> decryptor.decrypt(
                        encryptedPassword,
                        WRONG_KEY
                )
        );
    }

    @Test
    @DisplayName("Should reject a modified ciphertext")
    void shouldRejectModifiedCiphertext() throws Exception {
        String encryptedPassword =
                encrypt("Db@Password123", VALID_KEY);

        byte[] payload = Base64.getDecoder()
                .decode(encryptedPassword);

        payload[payload.length - 1] ^= 1;

        String modifiedPayload = Base64.getEncoder()
                .encodeToString(payload);

        assertThrows(
                DecryptionException.class,
                () -> decryptor.decrypt(
                        modifiedPayload,
                        VALID_KEY
                )
        );
    }

    private String encrypt(
            String plainText,
            String secretKey
    ) throws Exception {
        byte[] initializationVector = new byte[12];
        new SecureRandom().nextBytes(initializationVector);

        Cipher cipher =
                Cipher.getInstance("AES/GCM/NoPadding");

        SecretKeySpec keySpec = new SecretKeySpec(
                secretKey.getBytes(StandardCharsets.UTF_8),
                "AES"
        );

        GCMParameterSpec parameterSpec =
                new GCMParameterSpec(
                        128,
                        initializationVector
                );

        cipher.init(
                Cipher.ENCRYPT_MODE,
                keySpec,
                parameterSpec
        );

        byte[] cipherText = cipher.doFinal(
                plainText.getBytes(StandardCharsets.UTF_8)
        );

        byte[] payload = ByteBuffer
                .allocate(
                        initializationVector.length
                                + cipherText.length
                )
                .put(initializationVector)
                .put(cipherText)
                .array();

        return Base64.getEncoder().encodeToString(payload);
    }
}
```

## Assistant - Dependency Maven

```xml
<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.38</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.17</version>
    </dependency>

    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.5.18</version>
        <scope>runtime</scope>
    </dependency>

    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.12.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

# 4. Phân tích kịch bản What-if

## 4.1. Ciphertext null hoặc rỗng

Hàm dừng ngay tại bước validation và ném `DecryptionException`. Việc này tránh gọi Base64 decoder với dữ liệu không hợp lệ.

## 4.2. Secret key null hoặc rỗng

Hàm không tạo `SecretKeySpec` mà báo lỗi cấu hình ngay. Thông báo không chứa giá trị key.

## 4.3. Key sai độ dài

AES chỉ chấp nhận khóa 128, 192 hoặc 256 bit, tương ứng 16, 24 hoặc 32 byte. Hàm kiểm tra theo số byte UTF-8 trước khi gọi Cipher.

## 4.4. Sai secret key

Với AES-GCM, sai key thường làm xác thực tag thất bại và phát sinh `AEADBadTagException`, là lớp con của `BadPaddingException`. Hàm chuyển lỗi này thành `DecryptionException`.

## 4.5. Ciphertext bị sửa

GCM phát hiện thay đổi thông qua authentication tag. `doFinal` thất bại thay vì trả về plaintext sai.

## 4.6. InvalidKeyException

Lỗi được bắt riêng và log thông báo tổng quát. Không ghi key hoặc dữ liệu nhạy cảm.

## 4.7. Logging an toàn

Log chỉ mô tả nhóm lỗi:

- Invalid encoded payload.
- Authentication failed.
- Invalid cipher parameters.
- Invalid AES key.

Log không chứa:

- Mật khẩu gốc.
- Mật khẩu sau giải mã.
- Secret key.
- Toàn bộ ciphertext.
- Dữ liệu cấu hình nhạy cảm.

---

# 5. Lưu ý bảo mật thực tế

Trong hệ thống production, secret key không nên được hardcode trong source code hoặc commit vào Git. Nên lấy key từ:

- Secret manager.
- Biến môi trường được bảo vệ.
- Hardware Security Module.
- Dịch vụ quản lý khóa của nền tảng cloud.

Ngoài ra, ứng dụng nên có cơ chế xoay vòng khóa và quản lý phiên bản key. Phần test sử dụng key dạng chuỗi cố định chỉ nhằm mục đích minh họa.
