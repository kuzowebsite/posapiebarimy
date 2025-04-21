Ebarimt.mn Client API – OpenFeign ашиглан холбогдох загвар
1. Ашигласан эх сурвалжууд:
НӨАТ 0%-ийн бараа:
vat zero goods

НӨАТ-аас чөлөөлөгдсөн бараа:
vat free goods

Бараа ангиллын код:
gs1 classification

Сум, дүүргийн код:
District API

API баримт бичиг:

POS API 3.0.1

POS Combine.pdf

2. PosService суулгах заавар
2.1 Staging серверт хандалт шалгах
https://stg-invoice.ebarimt.mn/

https://st-operator.ebarimt.mn/

2.2 PosService суулгах
bash
Copy
Edit
sudo apt install unzip ar tar xz-utils
Туршилтын (staging) орчинд:

bash
Copy
Edit
curl -s https://raw.githubusercontent.com/hurelhuyag/ebarimt/master/install.sh | bash
Үйлдвэрлэл (prod) орчинд:

bash
Copy
Edit
curl -s https://raw.githubusercontent.com/hurelhuyag/ebarimt/master/install.sh | bash -s -- --prod
2.3 PosService тохируулах
http://127.0.0.1:7080 хаягаар нэвтэрч POS дугаарыг хуулж авна (8 оронтой).

st-operator.ebarimt.mn → POS дугаараар хайж, эхний нүд рүү хоёр даралт хийнэ.

"Мерчант нэмэх" товч → demo merchant 37900846788-аар хайж хадгална.

stg-invoice.ebarimt.mn → "Хүсэлт" → "Pos Api хүсэлт" → "+" → холболтоо бүртгүүлнэ.

"Баталгаажсан" гэсэн төлөв гарч ирэх ёстой.

3. API ашиглах заавар
3.1 pom.xml тохиргоо:
xml
Copy
Edit
<project>
    <properties>
        <spring-cloud.version>2022.0.3</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>io.github.hurelhuyag</groupId>
            <artifactId>ebarimt</artifactId>
            <version>3.0.1+2</version>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
3.2 application.properties файлд ebarimt service-ийн хаяг нэмэх:
properties
Copy
Edit
spring.cloud.openfeign.client.config.EbarimtApi.url=http://127.0.0.1:7080
3.3 OpenFeign client идэвхжүүлэх:
java
Copy
Edit
@SpringBootApplication
@EnableFeignClients({
    "io.github.hurelhuyag.ebarimt",
})
public class App {}
3.4 API-г дуудаж ebarimt үүсгэх:
java
Copy
Edit
@Service
@RequiredArgsConstructor
public class SimpleOrderService implements OrderService {

    private final EbarimtApi ebarimtApi;

    @Override
    public void createEbarimt(Long id) {
        var ebarimt = ebarimtApi.createReceipt(new CreateReceipt(
            new BigDecimal("112.00"),
            new BigDecimal("10.00"),
            new BigDecimal("2.00"),
            "3502",
            37900846788L,
            "123",
            "5678",
            null,
            null,
            CreateReceipt.Type.B2C_RECEIPT,
            null,
            null,
            List.of(
                new CreateReceipt.Receipt(
                    new BigDecimal("112.00"),
                    new BigDecimal("10.00"),
                    new BigDecimal("2.00"),
                    CreateReceipt.VatType.VAT_ABLE,
                    37900846788L,
                    null,
                    List.of(
                        new CreateReceipt.ReceiptItem(
                            "Хатуу чихэр",
                            "UNDEFINED",
                            CreateReceipt.BarCodeType.UNDEFINED,
                            2352010L,
                            null,
                            "p",
                            new BigDecimal("1.000"),
                            new BigDecimal("112.00"),
                            null,
                            new BigDecimal("10.00"),
                            new BigDecimal("2.00"),
                            new BigDecimal("112.00"),
                            null
                        )
                    )
                )
            ),
            List.of(
                new CreateReceipt.Payment(
                    CreateReceipt.PaymentCode.CASH,
                    CreateReceipt.PaymentStatus.PAID,
                    new BigDecimal("112.00"),
                    null
                )
            )
        ));
    }
}
