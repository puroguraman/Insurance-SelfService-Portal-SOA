**Έλεγχος Λογισμικού (Unit Testing)**



**package com.insurance.portal.service;**

**import org.junit.jupiter.api.Test;**

**import org.springframework.beans.factory.annotation.Autowired;**

**import org.springframework.boot.test.context.SpringBootTest;**

**import java.math.BigDecimal;**

**import java.util.List;**

**import static org.junit.jupiter.api.Assertions.assertEquals;**

**@SpringBootTest**

**class UnderwritingServiceTest {**

&#x20;**@Autowired**

&#x20;**private UnderwritingService underwritingService;**

&#x20;**@Test**

&#x20;**void testCalculateNewPremium() {**

&#x20;**// Σενάριο Δοκιμής:**

&#x20;**// Έστω ένα συμβόλαιο με ID=1 (Βασικό Ασφάλιστρο = 100.00€)**

&#x20;**// Ο χρήστης επιλέγει την κάλυψη με ID=2 (Οδική Βοήθεια =**

**30.00€)**



&#x20;**Long policyId = 1L;**

&#x20;**List<Long> selectedCoverageIds = List.of(2L);**

&#x20;**// Εκτέλεση της μεθόδου υπολογισμού**

&#x20;**BigDecimal result =**

**underwritingService.calculateNewPremium(policyId,**

**selectedCoverageIds);**

&#x20;**// Έλεγχος (Assertion): Το αποτέλεσμα πρέπει να είναι ακριβώς**

**130.00€**

&#x20;**assertEquals(new BigDecimal("130.00"), result, "Ο υπολογισμός**

**του ασφαλίστρου δεν είναι σωστός.");**

&#x20;**}**

**}**



