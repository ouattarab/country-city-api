--- Modèle Produit
export interface Produit {
  nom: string;
  prix: number;
  quantite: number;
}
----Service ProduitService
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { Produit } from '../models/produit.model';

@Injectable({
  providedIn: 'root',
})
export class ProduitService {
  private apiUrl = 'http://localhost:8080/api/produits';

  constructor(private http: HttpClient) {}

  enregistrerProduits(produits: Produit[]): Observable<string> {
    return this.http.post<string>(`${this.apiUrl}/enregistrer`, produits);
  }
}
----Composant ProduitComponent

import { Component, OnInit } from '@angular/core';
import { ProduitService } from '../services/produit.service';
import { Produit } from '../models/produit.model';

@Component({
  selector: 'app-produit',
  templateUrl: './produit.component.html',
  styleUrls: ['./produit.component.css'],
})
export class ProduitComponent implements OnInit {
  produits: Produit[] = [
    { nom: 'Produit A', prix: 25.5, quantite: 10 },
    { nom: 'Produit B', prix: 15.0, quantite: 20 },
    { nom: 'Produit C', prix: 30.0, quantite: 5 },
  ];

  constructor(private produitService: ProduitService) {}

  ngOnInit(): void {}

  enregistrerProduits(): void {
    this.produitService.enregistrerProduits(this.produits).subscribe({
      next: (message) => {
        console.log(message);
        alert('Produits enregistrés avec succès !');
      },
      error: (error) => {
        console.error('Erreur lors de l\'enregistrement', error);
        alert('Échec de l\'enregistrement');
      },
    });
  }
}
=======Backend
 ----Modèle Produit
 import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "produits")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Produit {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "produit_seq")
    @SequenceGenerator(name = "produit_seq", sequenceName = "produit_sequence", allocationSize = 1)
    private Long id;

    private String nom;
    private double prix;
    private int quantite;
}
----Service ProduitService
import org.springframework.stereotype.Service;
import lombok.RequiredArgsConstructor;
import java.util.List;

@Service
@RequiredArgsConstructor
public class ProduitService {

    private final ProduitRepository produitRepository;

    public void enregistrerProduits(List<Produit> produits) {
        produitRepository.saveAll(produits);
    }
}
----ProduitController.java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import lombok.RequiredArgsConstructor;
import java.util.List;

@RestController
@RequestMapping("/api/produits")
@RequiredArgsConstructor
public class ProduitController {

    private final ProduitService produitService;

    @PostMapping("/enregistrer")
    public ResponseEntity<String> enregistrerProduits(@RequestBody List<ProduitDTO> produitsDTO) {
        produitService.enregistrerProduits(produitsDTO);
        return ResponseEntity.ok("Produits enregistrés avec succès !");
    }
}

-------complet
Entité Produit.java

import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "produits")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Produit {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "produit_seq")
    @SequenceGenerator(name = "produit_seq", sequenceName = "produit_sequence", allocationSize = 1)
    private Long id;

    private String nom;
    private double prix;
    private int quantite;
}

✅ Utilisation de séquence PostgreSQL pour gérer l'auto-incrémentation.
2.2. DTO ProduitDTO.java

import lombok.*;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class ProduitDTO {
    private String nom;
    private double prix;
    private int quantite;
}

✅ DTO utilisé pour transférer les données depuis Angular vers Spring Boot.
2.3. Repository ProduitRepository.java

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ProduitRepository extends JpaRepository<Produit, Long> {
}

✅ Interface JPA pour les opérations en base.
2.4. Service ProduitService.java

import org.springframework.stereotype.Service;
import lombok.RequiredArgsConstructor;
import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
public class ProduitService {

    private final ProduitRepository produitRepository;

    public void enregistrerProduits(List<ProduitDTO> produitDTOs) {
        List<Produit> produits = produitDTOs.stream()
                .map(dto -> new Produit(null, dto.getNom(), dto.getPrix(), dto.getQuantite()))
                .collect(Collectors.toList());

        produitRepository.saveAll(produits);
    }
}

✅ Utilisation de saveAll() pour l'insertion en masse.
2.5. Contrôleur ProduitController.java

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import lombok.RequiredArgsConstructor;
import java.util.List;

@RestController
@RequestMapping("/api/produits")
@RequiredArgsConstructor
public class ProduitController {

    private final ProduitService produitService;

    @PostMapping("/enregistrer")
    public ResponseEntity<String> enregistrerProduits(@RequestBody List<ProduitDTO> produitsDTO) {
        produitService.enregistrerProduits(produitsDTO);
        return ResponseEntity.ok("Produits enregistrés avec succès !");
    }
}
