<script>
  import Notiflix from "notiflix";
  //Importaciones para db
  import { db } from "../firebase";
  import {
    addDoc,
    collection,
    onSnapshot,
    deleteDoc,
    doc,
    updateDoc,
    QuerySnapshot,
  } from "firebase/firestore";

  import { onDestroy } from "svelte";

  let menu = "Prestamo de cartas";
  let cartas = [];
  let loans = [];
  let vista = 0;

  function handle(e){

    let a = e.target.parentNode.children[0].id;
    let b = e.target.parentNode.children[1].id; 

    const btna = document.getElementById(a);
    const btnb = document.getElementById(b);

    btna.disabled = true;
    btnb.disabled = true;
  }

  const onsub = onSnapshot(
    collection(db, "cartaBiblioteca"),
    (QuerySnapshot) => {
      cartas = QuerySnapshot.docs.map((doc) => {
        return { ...doc.data(), id: doc.id };
      });
    },
    (err) => {
      console.log(err);
    }
  );

  const onsub1 = onSnapshot(
    collection(db, "loans"),
    (QuerySnapshot) => {
      loans = QuerySnapshot.docs.map((doc) => {
        return { ...doc.data(), id: doc.id };
      });
    },
    (err) => {
      console.log(err);
    }
  );

  async function pedirPrestamo(ID) {
    let cantidad = prompt(
      "Por favor, ingresa la cantidad de préstamos (1, 2 o 3)",
      "1"
    );
    try {
      const nuevoPrestamo = {
        nombrecarta: cartas.find((carta) => carta.id === ID).nombre,
        cantidad: cantidad,
        cartaID: ID,
        usuario: "admin",
        fecha: new Date().toLocaleDateString(),
        estado: "Pendiente",
      };
      let total = parseInt(cartas.find((carta) => carta.id === ID).prestadas) + parseInt(cantidad)
      if (cantidad > cartas.find((carta) => carta.id === ID).cantidad) {
        throw new Error("No hay suficientes cartas en la biblioteca");
      } else if (cantidad < 1 || cantidad > 3) {
        throw new Error("Cantidad no válida");
      }else if( total > parseInt(cartas.find((carta) => carta.id === ID).cantidad)){
        throw new Error("No hay suficientes cartas en la biblioteca");
      }
      await addDoc(collection(db, "loans"), nuevoPrestamo);
      Notiflix.Notify.success("Solicitud de préstamo enviada");
    } catch (e) {
      Notiflix.Notify.failure(
        "Solicitud de préstamo cancelada por " + e.message
      );
    }
  }

  onDestroy(onsub);
  onDestroy(onsub1);

  function AcepRech(ID, estado, prestadoa, cartaID) {
    updateDoc(doc(db, "loans", ID), {
      estado: estado,
    });
    let total = (parseInt(prestadoa)+ parseInt(cartas.find((carta) => carta.id === cartaID).prestadas));
    updateDoc(doc(db, "cartaBiblioteca", cartaID), {
      prestadas: total,
    });
    
  }

  function cambiarvista(x) {
    if (x == 0) {
      vista = 0;
    } else {
      vista = 1;
    }
  }
</script>

<body id="page-top">
  <!-- Page Wrapper -->
  <div id="wrapper">
    <!-- Sidebar -->
    <ul class="navbar-nav sidebar sidebar-dark accordion" id="accordionSidebar">
      <!-- Sidebar - Brand -->
      <a
        class="sidebar-brand d-flex align-items-center justify-content-center"
        href="/cartas"
      >
        <div class="sidebar-brand-icon mt-4">
          <img
            src="https://firebasestorage.googleapis.com/v0/b/omegaproxy-4abfe.appspot.com/o/isotipo.png?alt=media&token=55b51f65-c906-4427-ba2b-39bfea66ada9"
            alt="Descripción de la imagen"
            style="width: 120px;"
          />
        </div>
      </a>
      <br />
      <!-- Divider -->
      <hr class="sidebar-divider my-0" />

      <!-- Divider -->
      <hr class="sidebar-divider" />

      <!-- Heading -->
      <div
        style="font-size:larger; color:white; text-align:center"
        class="sidebar-heading"
      >
        Menu
      </div>

      <!-- Nav Item - Charts -->
      <li class="nav-item">
        <a class="nav-link" href="/cartas">
          <i class="fas fa-fw fa-chart-area"></i>
          <span style="font-size:larger;">Cartas</span></a
        >
      </li>

      <!-- Nav Item - Charts -->
      <li class="nav-item">
        <a class="nav-link" href="/dame">
          <i class="fas fa-fw fa-chart-area"></i>
          <span style="font-size:larger;">Prestamos</span></a
        >
      </li>

      <!-- Nav Item - Tables -->
      <li class="nav-item">
        <a class="nav-link" href="/jugadores">
          <i class="fas fa-fw fa-clipboard-list"></i>
          <span style="font-size:larger;">Jugadores</span></a
        >
      </li>

      <!-- Nav Item - Charts -->
      <li class="nav-item">
        <a class="nav-link" href="/records">
          <i class="fas fa-fw fa-chart-area"></i>
          <span style="font-size:larger;">Records</span></a
        >
      </li>

      <!-- Divider -->
      <hr class="sidebar-divider d-none d-md-block" />
    </ul>
    <!-- End of Sidebar -->

    <!-- Content Wrapper -->
    <div id="content-wrapper" class="d-flex flex-column">
      <!-- Main Content -->
      <div id="content">
        <!-- Topbar -->
        <nav
          class="navbar navbar-expand navbar-light bg-white topbar mb-4 static-top shadow"
        >
          <!-- Sidebar Toggle (Topbar) -->
          <button
            id="sidebarToggleTop"
            class="btn btn-link d-md-none rounded-circle mr-3"
          >
            <i class="fa fa-bars"></i>
          </button>

          <!-- Topbar Navbar -->
          <ul class="navbar-nav ml-auto">
            <li class="nav-item mt-2">
              <img
                src="https://firebasestorage.googleapis.com/v0/b/omegaproxy-4abfe.appspot.com/o/logotipo.png?alt=media&token=8e737513-5251-42be-b83f-efaeb59edf79"
                alt="Navbar Logo"
                class="img-fluid"
                width="200px"
                style="margin-top: 10px;"
              />
            </li>
            <!-- User Info -->
            <li class="nav-item dropdown no-arrow">
              <a
                class="nav-link dropdown-toggle"
                href="/"
                id="userDropdown"
                role="button"
                data-toggle="dropdown"
                aria-haspopup="true"
                aria-expanded="false"
              >
                <span class="mr-2 d-none d-lg-inline text-gray-600 small"
                ></span>

                <button type="button" class="btn btn-outline-danger">
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width="16"
                    height="16"
                    fill="currentColor"
                    class="bi bi-box-arrow-right"
                    viewBox="0 0 16 16"
                  >
                    <path
                      fill-rule="evenodd"
                      d="M10 12.5a.5.5 0 0 1-.5.5h-8a.5.5 0 0 1-.5-.5v-9a.5.5 0 0 1 .5-.5h8a.5.5 0 0 1 .5.5v2a.5.5 0 0 0 1 0v-2A1.5 1.5 0 0 0 9.5 2h-8A1.5 1.5 0 0 0 0 3.5v9A1.5 1.5 0 0 0 1.5 14h8a1.5 1.5 0 0 0 1.5-1.5v-2a.5.5 0 0 0-1 0z"
                    ></path>
                    <path
                      fill-rule="evenodd"
                      d="M15.854 8.354a.5.5 0 0 0 0-.708l-3-3a.5.5 0 0 0-.708.708L14.293 7.5H5.5a.5.5 0 0 0 0 1h8.793l-2.147 2.146a.5.5 0 0 0 .708.708z"
                    ></path>
                  </svg>
                  Salir
                </button>
              </a>
              <!-- logout -->
              <div
                class="dropdown-menu dropdown-menu-right shadow animated--grow-in"
                aria-labelledby="userDropdown"
              >
                <a
                  class="dropdown-item"
                  href="/"
                  data-toggle="modal"
                  data-target="#logoutModal"
                >
                  <i class="fas fa-sign-out-alt fa-sm fa-fw mr-2 text-gray-400"
                  ></i>
                  Logout
                </a>
              </div>
            </li>
          </ul>
        </nav>
        <!-- End of Topbar -->

        <!-- Begin Page Content -->
        <div class="container-fluid">
          <!-- Page Heading -->
          <div
            class="d-sm-flex align-items-center justify-content-between mb-4"
          >
            <h1 class="h3 mb-0 text-gray-800">{menu}</h1>
          </div>
          <button class="btn btn-primary" on:click={() => cambiarvista(0)}>Pedir Prestamo</button>
          <button class="btn btn-primary" on:click={() => cambiarvista(1)}>Administrar Prestamos</button
          >
        </div>
        <!-- /.container-fluid -->
        {#if vista == 0}
          <table
            id="Table"
            class="table table-striped m-3"
            style="width:100%"
            data-toggle="table"
            data-seach="true"
            data-searchable="true"
            data-pagination="true"
          >
            <thead>
              <tr>
                <th>Nombre de la carta</th>
                <th>Tipo</th>
                <th>Cantidad en biblioteca</th>
                <th>ID</th>
                <th>Acción</th>
              </tr>
            </thead>
            {#each cartas as carta (carta.id)}
              <tbody>
                <tr>
                  <td>{carta.nombre}</td>
                  <td>{carta.tipo}</td>
                  <td>{carta.cantidad}</td>
                  <td>{carta.id}</td>
                  <td>
                    <button class="btn btn-primary m-1 p-0" on:click={() => pedirPrestamo(carta.id)}
                      >Pedir prestamo</button
                    >
                  </td>
                </tr>
              </tbody>
            {/each}
          </table>
        {/if}
        {#if vista == 1}
          <table
            id="Table"
            class="table table-striped"
            style="width:100%"
            data-toggle="table"
            data-seach="true"
            data-searchable="true"
            data-pagination="true"
          >
            <thead>
              <tr>
                <th>Nombre de la carta</th>
                <th>Cantidad a prestamo</th>
                <th>ID de la carta</th>
                <th>Fecha de la solicitud</th>
                <th>Estado</th>
                <th>Usuario</th>
                <th>Aceptar/Rechazar</th>
              </tr>
            </thead>
            {#each loans as loan, num (loan.id)}
              <tbody>
                <tr>
                  <td>{loan.nombrecarta}</td>
                  <td>{loan.cantidad}</td>
                  <td>{loan.cartaID}</td>
                  <td>{loan.fecha}</td>
                  <td>{loan.estado}</td>
                  <td>{loan.usuario}</td>
                  <td>
                    <button id = {"btna"+num} on:click={handle} on:click={() => AcepRech(loan.id, "Aceptado", loan.cantidad, loan.cartaID)}
                      >Aceptar</button
                    >
                    <button id = {"btnb"+num} on:click={handle} on:click={() => AcepRech(loan.id, "Rechazado", loan.cantidad, loan.cartaID)}
                      >Rechazar</button
                    >
                </tr>
              </tbody>
            {/each}
          </table>
        {/if}

        <!-- End of Main Content -->
        <!-- End of Footer -->
      </div>
      <!-- End of Content Wrapper -->
    </div>
    <!-- End of Page Wrapper -->
  </div>
</body>

<style>
  .sidebar {
    background-color: rgb(242, 91, 44);
  }
  button:hover {
    cursor: pointer;
    background-color: rgb(175, 123, 45);
    color: black;
  }
</style>
