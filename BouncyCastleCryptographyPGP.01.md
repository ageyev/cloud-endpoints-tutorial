В данном руководстве мы рассмотрим работу с OpenPGP на Java с использованием библиотеки  Bouncy Castle Cryptography Library с ориентацией на использование в веб-разработке.

<img src="https://habrastorage.org/getpro/habr/post_images/bbe/0f7/ebd/bbe0f7ebd2390f418eae9bda5f286e96.png" alt="image"/>
<cut />
(<a href="https://commons.wikimedia.org/wiki/File:Digital_Signature_diagram_ru.png?uselang=ru">ссылка на источник схемы</a>)

<h4><a href="https://ru.wikipedia.org/wiki/PGP">PGP</a>/<a href="https://ru.wikipedia.org/wiki/PGP#OpenPGP">OpenPGP</a></h4>
<a href="https://ru.wikipedia.org/wiki/PGP#OpenPGP">OpenPGP</a> описан в  <a href="https://tools.ietf.org/html/rfc4880">RFC 4880</a>, и является наиболее распространенным стандартом <a href="https://ru.wikipedia.org/wiki/%D0%9A%D1%80%D0%B8%D0%BF%D1%82%D0%BE%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D0%B0_%D1%81_%D0%BE%D1%82%D0%BA%D1%80%D1%8B%D1%82%D1%8B%D0%BC_%D0%BA%D0%BB%D1%8E%D1%87%D0%BE%D0%BC">криптографической системы с открытым ключем</a> для работы с цифровыми подписями и шифрованием сообщений. Поддерживается множеством программ и библиотек, в том числе с свободных с отрытым кодом. В частности, следует отметить <a href="https://www.gnupg.org/">GnuPG</a>, <a href="https://www.kde.org/applications/utilities/kleopatra/">Kleopatra</a>, набор утилит для Windows <a href="https://www.gpg4win.org/">Gpg4win</a> (в него входят GnuPG, Kleopatra и др.), расширение для Thunderbird <a href="https://www.enigmail.net">Enigmail</a>.
Для Windows и  также следует обратить внимание на коммерческий пакет Symantec Endpoint Encryption, известный также как PGP Desktop (доступна <a href="https://www4.symantec.com/Vrt/offer?a_id=184016">пробная версия</a>, которую после истечения срока можно продолжать использовать для некоммерческих целей), и другие продукты из <a href="https://www.symantec.com/encryption/">Symantec Encryption Family</a>.

Подробнее см.: <a href="https://ssd.eff.org/ru/module/%D0%B2%D0%B2%D0%B5%D0%B4%D0%B5%D0%BD%D0%B8%D0%B5-%D0%B2-%D1%88%D0%B8%D1%84%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D1%81-%D0%BE%D1%82%D0%BA%D1%80%D1%8B%D1%82%D1%8B%D0%BC-%D0%BA%D0%BB%D1%8E%D1%87%D0%BE%D0%BC-%D0%B8-pgp">Введение в шифрование с открытым ключом и PGP</a>, <a href="https://ru.wikipedia.org/wiki/PGP#.D0.9C.D0.B5.D1.85.D0.B0.D0.BD.D0.B8.D0.B7.D0.BC_.D1.80.D0.B0.D0.B1.D0.BE.D1.82.D1.8B_PGP">Механизм работы PGP</a>.

<h4>Bouncy Castle Cryptography Library</h4>
Bouncy Castle Cryptography Library в настоящее время является наиболее используемой криптографической библиотекой для Java, в том чиле <a href="https://en.wikipedia.org/wiki/Bouncy_Castle_%28cryptography%29#Spongy_Castle">используется</a> в Android.
Это проект с открытым кодом (GitHub: https://github.com/bcgit/bc-java ). Также для нашей темы интерес представляет открытый код программы <a href="http://ppgp.sourceforge.net/">Portable PGP</a> v. 1.x , написанной на Java c использованием Bouncy Castle: http://sourceforge.net/p/ppgp/code/

Для работы с OpenPGP в Bouncy Castle есть отдельный пакет <a href="https://www.bouncycastle.org/docs/pgdocs1.5on/index.html">org.bouncycastle.openpgp</a>, с которым мы и будем работать.

<h4>Считываем публичный ключ</h4>
Решаемая задача: считать ASCII-armored Public PGP Key Block (публичный ключ) в виде String и создать <a href="https://www.bouncycastle.org/docs/pgdocs1.5on/org/bouncycastle/openpgp/PGPPublicKey.html">org.bouncycastle.openpgp.PGPPublicKey</a> объект, представляющий публичный ключ в BC.

ASCII-armored формат представляет ключ текстовом формате, который удобно использовать для публикации на сайте, пересылки по электронной почте и т.д., выглядит так:
<source lang="xml">
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v2.0.22 (GNU/Linux)

mQINBFZpf0wBEADo7NZ4Oymw2pVJMfrNEhGwYOT4CGMOTZHQo3rYFrN3yxCsd/xX
r8kWLvkKvEFSD0XfQlq6slA9fnOtsRZl4JlmabKC33ZkB1zhV2c78AhhVMrfFi2a
114xHhOHkR4LOv8mAyyRKd5mpuyYpPKcF2670jXxANeqNQCoKKM/dPS1uxGapE9Z
GG0GFKvIUqKWJUAv7JqOAxtXbAS7JFMwiH16jL/TeuvKy+0JKvlBM4qxB9S18hi4
GYg1SEATqmeu9E+6alz0L25REYYZZOMja1quF8HywsRY0fSZqCbD+9diP0keAg6z
PSDSlgDF4WBUi+c8PoXcRRZq7+XYgky4l8wfGoGnOZKA4GH2SMlNAX8jCIrC7Cpf
KPCu6NMY533X735Md6fnWOeuyxz4owPRb6OTt4rYiA3/9vCu/waZ1zZx/wATYORS
1wmOx8ZjogeAPOz6a2PW8hrK0tWbT3ILucC7dxjYfdKHZX2+v3eMGvXFRKss8J5i
E5rlJiDnAMoERmIJunh/EJHz4te+RdMy2jWeDQWGzaFyZC92SUo9tAUJk7xez0KX
5sYzhXjFJX/17DwFBTXIOYpFjWUyPaQaP7ByWOjZzhCmQxYRBUJSoxOty1ngkZ+B
7zAZSTXh2mukHukRERG1//2FsiBZeapRhAaFWdWnPDN95bg7L7/qQ5szjQARAQAB
tDdWaWt0b3IgQWdleWV2IDxhZ2V5ZXZAaW50ZXJuYXRpb25hbC1hcmJpdHJhdGlv
bi5vcmcudWs+iQI/BBMBAgApBQJWaX9MAhsvBQkB4lLUBwsJCAcDAgEGFQgCCQoL
BBYCAwECHgECF4AACgkQzjaf2edxc+UywQ/+MfYX5BRzl7iSndhr/vVOEycxZntu
9efMhwQYO87EXT/cNpRMX/u2Wzhx70brzFIenvaYTPpVkYKinKv3iEAauJ1wr5/a
PrUxRcfVnBrV8ecWTO6SbNRDePqayst8bBq9rdqnKnpDEv911zk2hjJtCeFDlUkO
eiTLgYCVOQ7n0fbN4pjrBGAqDGs2SFjJXAKYJpZqY77P0crHxXCMyukrdj2WxnB7
JyMdmBqViJSo/MCi7SFXqVQFTvJZVBhTQEUO5KNJA6oAYztYKXBE3AybE528m571
p+HhJ2mOnlNaRu5y12RRQJ7qmKvN9uzHz0+mt8rRk3oYWCYImTMwxDpc1mT39QKs
0WS/zp5NTIaa+Hvt7V8GH5hmq67YL+JcxEINZGZ+MZmiSKuvGVuqC8ZJpswumVw5
PHtMEL+0zBxZXN41YyfTudssXbvMrNiGIdlQA7CW/7yMviVR4nyQtFQJwYSL4mva
uTQSnihm4Bj6FeYpgGpk1TyIzgtIyaQNSvnlDYFezmN/JsAs+25EP/oxyFLaK+ij
WdjVmnsDGS4l5BWH/a8BYPbSnm7YrlYkYCXjTjK1/EeWA/tggApISFptc8YQw0Wp
Qw0IwhJVPWDVW8nDrv/1IPhTJgYfIfdmxr32RLhNePEZGZ02zfCP9BrlNKut9/Hn
x1yKWjVaWZO8VtU=
=XyhJ
-----END PGP PUBLIC KEY BLOCK-----
</source>
В таком виде публичный ключ может быть передан в веб-форму. Мы напишем код который может на сервере извлечь данные из ключа, и затем, скажем, вернуть их пользователю (сервис распознавания публичных ключей) и/или сохранить в базе данных. В BC есть класс <a href="https://www.bouncycastle.org/docs/pgdocs1.5on/org/bouncycastle/openpgp/PGPPublicKey.html">PGPPublicKey</a>, нашей задачей будет создать новый объект этого класса используя на входе String , а потом используя методы предоставляемые этим классом получить данные о ключе.

В пакете org.bouncycastle.openpgp.examples есть класс PGPExampleUtil , мы не сможем его импортировать так как он непубличный, но его код можем использовать для создания своего аналогичного класса. Метод из PGPExampleUtil который мы можем взять за основу:
<source lang="java">
/**
     * A simple routine that opens a key ring file and loads the first available key
     * suitable for encryption.
     *
     * @param input data stream containing the public key data
     * @return the first public key found.
     * @throws IOException
     * @throws PGPException
     */
    static PGPPublicKey readPublicKey(InputStream input) throws IOException, PGPException
    {
        PGPPublicKeyRingCollection pgpPub = new PGPPublicKeyRingCollection(
            PGPUtil.getDecoderStream(input), new JcaKeyFingerprintCalculator());

        //
        // we just loop through the collection till we find a key suitable for encryption, in the real
        // world you would probably want to be a bit smarter about this.
        //

        Iterator keyRingIter = pgpPub.getKeyRings();
        while (keyRingIter.hasNext())
        {
            PGPPublicKeyRing keyRing = (PGPPublicKeyRing)keyRingIter.next();

            Iterator keyIter = keyRing.getPublicKeys();
            while (keyIter.hasNext())
            {
                PGPPublicKey key = (PGPPublicKey)keyIter.next();

                if (key.isEncryptionKey())
                {
                    return key;
                }
            }
        }

        throw new IllegalArgumentException("Can't find encryption key in key ring.");
    }

</source>
В <a href="http://sourceforge.net/p/ppgp/code/HEAD/tree/trunk/src/portablepgp/core/KeyRing.java">portablepgp.core.KeyRing</a> похожая задача решается следующим образом:
<source lang="java">
    public static PGPPublicKeyRing importPublicKey(InputStream iKeyStream) throws IOException {

        PGPPublicKeyRing newKey = new PGPPublicKeyRing(new ArmoredInputStream(iKeyStream));

        pubring = PGPPublicKeyRingCollection.addPublicKeyRing(pubring, newKey);
        savePubring();

        return newKey;
    }
</source>
Исходя из предположения что нам нужно вынуть только один (первый) PGPPublicKey из PGPPublicKeyRing просто используем <a href="https://www.bouncycastle.org/docs/pgdocs1.5on/org/bouncycastle/openpgp/PGPPublicKeyRing.html#getPublicKey%28%29">.getPublicKey()</a> и не используем итератор (на входе String, метод возвращает org.bouncycastle.openpgp.PGPPublicKey) :
<source lang="java">
    public static PGPPublicKey importPublicKey (String armoredPublicPGPkeyBlock) throws IOException {

        InputStream in = new ByteArrayInputStream(armoredPublicPGPkeyBlock.getBytes());

        PGPPublicKeyRing pgpPublicKeyRing = new PGPPublicKeyRing(new ArmoredInputStream(in), new JcaKeyFingerprintCalculator());

        in.close();

        PGPPublicKey pgpPublicKey = pgpPublicKeyRing.getPublicKey();

        return pgpPublicKey;
    }
</source>
Либо мы можем полностью скопировать в наш класс метод readPublicKey  из org.bouncycastle.openpgp.examples.PGPExampleUtil , и дополнить класс следующим методом:
<source lang="java">
    public static PGPPublicKey readPublicKeyFromString (String armoredPublicPGPkeyBlock) throws IOException, PGPException {

        InputStream in = new ByteArrayInputStream(armoredPublicPGPkeyBlock.getBytes());

        PGPPublicKey pgpPublicKey = readPublicKey(in);

        in.close();

        return pgpPublicKey;
    }
</source>
<h4>Отображение данных ключа</h4>
На основе <a href="https://www.bouncycastle.org/docs/pgdocs1.5on/org/bouncycastle/openpgp/PGPPublicKey.html">org.bouncycastle.openpgp.PGPPublicKey</a> объекта создаем представление данных ключа, которое может быть прочитано человеком или сохранено в базе данных.

Для простейшего варианта представления данных публичного ключа в читаемом виде можно воспользоваться классом <a href="http://sourceforge.net/p/ppgp/code/HEAD/tree/trunk/src/portablepgp/core/PrintablePGPPublicKey.java">portablepgp.core.PrintablePGPPublicKey</a>  c методом toString() :
<source lang="java">
package portablepgp.core;

import java.util.Iterator;
import org.bouncycastle.openpgp.PGPPublicKey;

/**
 *
 * @author Primiano Tucci - http://www.primianotucci.com/
 */
public class PrintablePGPPublicKey  {
    PGPPublicKey base;

    public PrintablePGPPublicKey(PGPPublicKey iBase){
        base = iBase;
    }

    public PGPPublicKey getPublicKey(){
        return base;
    }

    @Override
    public String toString() {
        StringBuilder outStr = new StringBuilder();
        Iterator iter = base.getUserIDs();

        outStr.append("[0x");
        outStr.append(Integer.toHexString((int)base.getKeyID()).toUpperCase());
        outStr.append("] ");

        while(iter.hasNext()){
            outStr.append(iter.next().toString());
            outStr.append("; ");
        }

        return outStr.toString();
    }

}
</source>
Для хранения более полной информации о ключе создадим класс PGPPublicKeyData с конструктором извлекающим данные из <a href="https://www.bouncycastle.org/docs/pgdocs1.5on/org/bouncycastle/openpgp/PGPPublicKey.html">org.bouncycastle.openpgp.PGPPublicKey</a>:
<source lang="java">

import com.google.common.base.Splitter;
import com.google.gson.Gson;
import org.bouncycastle.openpgp.PGPPublicKey;

import javax.xml.bind.DatatypeConverter;
import java.util.Date;
import java.util.Iterator;
import java.util.List;

public class PGPPublicKeyData {
    String keyID;
    String userID;
    String firstName;
    String lastName;
    String userEmail;
    Date created;
    Date exp;
    int bitStrength;
    String asciiArmored;
    String fingerprint;

    public PGPPublicKeyData(PGPPublicKey pgpPublicKey) {
        // keyID
        StringBuilder keyIDstrBuilder = new StringBuilder();
        keyIDstrBuilder.append("[0x");
        keyIDstrBuilder.append(Integer.toHexString((int) pgpPublicKey.getKeyID()).toUpperCase());
        keyIDstrBuilder.append("] ");
        this.keyID = keyIDstrBuilder.toString();
        // userID
        StringBuilder userIDstrBuilder = new StringBuilder();
        Iterator userIDsIterator = pgpPublicKey.getUserIDs();
        while (userIDsIterator.hasNext()) {
            userIDstrBuilder.append(userIDsIterator.next().toString());
            userIDstrBuilder.append("; ");
        }
        this.userID = userIDstrBuilder.toString();
        // user's first and last name and email
        List<String> userIdList = Splitter.on(' ').trimResults().omitEmptyStrings().splitToList(userID);
        this.firstName = userIdList.get(0);
        this.lastName = userIdList.get(1);
        String userEmailDirty = userIdList.get(userIdList.size() - 1);
        this.userEmail = userEmailDirty.substring(
                1, userEmailDirty.length() - 2
        );
        // fingerprint // [0xE77173E5]
        this.fingerprint = DatatypeConverter.printHexBinary(
                pgpPublicKey.getFingerprint()
        );
        // creation date
        this.created = pgpPublicKey.getCreationTime();
        // exp date
        this.exp = org.apache.commons.lang3.time.DateUtils.addSeconds(created, (int) pgpPublicKey.getValidSeconds());
        // Bit strength
        this.bitStrength = pgpPublicKey.getBitStrength();
    }

    public String toJSON() {
        Gson gson = new Gson();
        return gson.toJson(this);
    }

    //    ---------- Getters and Setters:

}
</source>

<i>Продолжение следует</i>
