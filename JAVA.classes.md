1) **Κλάση Υπηρεσίας (Service Layer)**

*package com.insurance.portal.service;*

*import com.insurance.portal.model.Policy;*

*import com.insurance.portal.model.Coverage;*

*import com.insurance.portal.repository.PolicyRepository;*

*import com.insurance.portal.repository.CoverageRepository; import*

*org.springframework.beans.factory.annotation.Autowired;*

*import org.springframework.stereotype.Service;*

*import java.math.BigDecimal;*

*import java.util.List;*

*@Service*

*public class UnderwritingService {*

*@Autowired*

*private PolicyRepository policyRepo;*

*@Autowired*



*private CoverageRepository coverageRepo;*



*/\*\**

*\* Υπολογίζει το νέο ασφάλιστρο βάσει των επιλεγμένων καλύψεων.*

*\* @param policyId Το ID του συμβολαίου*

*\* @param coverageIds Λίστα με τα IDs των καλύψεων που επέλεξε ο*

*πελάτης*

*\* @return Το συνολικό ποσό πληρωμής*

*\*/*

*public BigDecimal calculateNewPremium(Long policyId, List<Long>*

*coverageIds) {*

*// 1. Ανάκτηση του συμβολαίου από τη Βάση*

*Policy policy = policyRepo.findById(policyId)*

*.orElseThrow(() -> new RuntimeException("Policy not found"));*

*// 2. Αρχικοποίηση με το βασικό ασφάλιστρο*

*BigDecimal totalPremium = policy.getBasePremium();*

*// 3. Προσθήκη κόστους για κάθε επιλεγμένη κάλυψη*

*List<Coverage> selectedCoverages =*

*coverageRepo.findAllById(coverageIds);*



*for (Coverage cov : selectedCoverages) {*

*totalPremium = totalPremium.add(cov.getCost()); }*

*return totalPremium;*

*}*





/\*\*

\* Ενημερώνει το συμβόλαιο με το νέο ασφάλιστρο. \*/

public void updatePolicy(Long policyId, BigDecimal newPremium) {

Policy policy = policyRepo.findById(policyId).get();

policy.setTotalPremium(newPremium);

policyRepo.save(policy);

}

}







**2) -Κλάση Ελέγχου (REST Controller)**

Αυτή η κλάση δέχεται τα αιτήματα από το Web Portal (Frontend).

package com.insurance.portal.controller;



import com.insurance.portal.service.UnderwritingService; import

org.springframework.beans.factory.annotation.Autowired; import

org.springframework.web.bind.annotation.\*;

import java.math.BigDecimal;

import java.util.List;

import java.util.Map;

@RestController

@RequestMapping("/api/policy")

public class PolicyController {

@Autowired

private UnderwritingService underwritingService;

// Endpoint: POST /api/policy/calculate

@PostMapping("/calculate")

public Map<String, Object> calculatePremium(@RequestBody Map<String,

Object> payload) {



// Λήψη δεδομένων από το JSON αίτημα

Long policyId = Long.valueOf(payload.get("policyId").toString());

&#x20;List<Long> coverageIds = (List<Long>) payload.get("coverageIds");

// Κλήση της υπηρεσίας υπολογισμού

BigDecimal newPremium =

underwritingService.calculateNewPremium(policyId, coverageIds);

// Επιστροφή αποτελέσματος σε JSON

return Map.of(

"status", "success",

"newPremium", newPremium,

"message", "Ο υπολογισμός ολοκληρώθηκε επιτυχώς." );

}

}





**3) - Υλοποίηση Ασφάλειας και Εξουσιοδότησης (Spring Security)** 

package com.insurance.portal.config;

import org.springframework.context.annotation.Bean;

import org.springframework.context.annotation.Configuration;

import

org.springframework.security.config.annotation.web.builders.HttpSecur

ity;

import

org.springframework.security.config.annotation.web.configuration.Enab

leWebS ecurity;

import org.springframework.security.web.SecurityFilterChain; import

org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration

@EnableWebSecurity

public class SecurityConfig {

@Bean

public SecurityFilterChain securityFilterChain(HttpSecurity http)

throws Exception {

http

.csrf().disable() // Απενεργοποίηση για απλότητα στο demo

.authorizeHttpRequests((requests) -> requests

// Δημόσια προσβάσιμες σελίδες (Login, Register, CSS/JS)

.requestMatchers("/", "/login", "/register", "/css/\*\*",

"/js/\*\*").permitAll()

// Σελίδες που απαιτούν ρόλο Πελάτη

.requestMatchers("/dashboard/\*\*",

"/api/policy/\*\*").hasRole("CUSTOMER")

// Σελίδες διαχειριστών (για μελλοντική χρήση)

.requestMatchers("/admin/\*\*").hasRole("ADMIN")

// Όλα τα άλλα απαιτούν απλή σύνδεση

.anyRequest().authenticated()

)

.formLogin((form) -> form

.loginPage("/login")

.defaultSuccessUrl("/dashboard", true)

.permitAll()

)

.logout((logout) -> logout.permitAll());

return http.build();

}

@Bean

public PasswordEncoder passwordEncoder() {

// Χρήση αλγορίθμου BCrypt για ασφαλή hashing κωδικών

return new BCryptPasswordEncoder();

}

}





**4) -Διαχείριση Εξαιρέσεων (Global Exception Handling**)

Για τη διασφάλιση της ομαλής λειτουργίας του συστήματος, υλοποιήθηκε

μηχανισμός καθολικής διαχείρισης σφαλμάτων μέσω της Annotation

@ControllerAdvice. Αυτό επιτρέπει στο σύστημα να εμφανίζει φιλικά μηνύματα προς

τον χρήστη αντί για τεχνικά μηνύματα λάθους (Stack Traces), όταν προκύπτουν

εξαιρέσεις.



package com.insurance.portal.exception;

import org.springframework.web.bind.annotation.ControllerAdvice;

import org.springframework.web.bind.annotation.ExceptionHandler;

import org.springframework.web.servlet.ModelAndView;

@ControllerAdvice

public class GlobalExceptionHandler {

// Χειρισμός περίπτωσης που δεν βρέθηκε εγγραφή (π.χ. λάθος ID

συμβολαίου) @ExceptionHandler(ResourceNotFoundException.class)

public ModelAndView handleNotFound(ResourceNotFoundException ex) {

ModelAndView mav = new ModelAndView("error-page");

mav.addObject("errorMessage", "Η εγγραφή που αναζητάτε δεν

βρέθηκε.");

return mav;

}



**// Χειρισμός γενικών τεχνικών σφαλμάτων** (π.χ. Database connection

error) @ExceptionHandler(Exception.class)

public ModelAndView handleGeneralError(Exception ex) {

ModelAndView mav = new ModelAndView("error-page");

mav.addObject("errorMessage", "Προέκυψε ένα απροσδόκητο

σφάλμα. Παρακαλώ δοκιμάστε αργότερα.");

return mav;

}

}





