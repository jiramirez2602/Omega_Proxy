<script>
    import Notiflix from "notiflix";
    import { goto } from "$app/navigation";
    import { admin, usuario } from "../lib/store/Store.js"
  
    let mensaje = "Por favor introduce tu usuario y contraseña!";
    let username = "";
    let password = "";
  
    import { db } from "./firebase";
    import { collection, onSnapshot } from "firebase/firestore";
    import { onDestroy } from "svelte";
  
    //Variable vacia donde se guardan los usuarios
    let users = [];
  
    //Actualizaciones de data en timpo real
    const onsub = onSnapshot(
      collection(db, "users"),
      (querySnapshot) => {
        users = querySnapshot.docs.map((docs) => {
          return { ...docs.data(), id: docs.id };
        });
        console.log(users);
      },
      (err) => {
        console.log(err);
      }
    );
  
    //Dejar de solicitar actualización al salir de ventana
    onDestroy(onsub);
  
    function login() {
      let comparador = users.find((usuario) => usuario.username === username);
      // Aquí puedes agregar la lógica para el inicio de sesión
      if (username === "admin" && password === "admin") {
        Notiflix.Notify.success("Bienvenido, has iniciado sesión como admin!");
        cambiarUsuario("Admin");
        esAdmin(1);
        goto("/cartas");
      } else if (comparador && comparador.password == password) {
        Notiflix.Notify.success("Bienvenido, has iniciado sesión como jugador!");
        cambiarUsuario(comparador.username);
        goto("/cartas");
        esAdmin(0);
      } else {
        Notiflix.Notify.failure("Usuario o contraseña inválida!");
      }
      
    }

    function cambiarUsuario(nuevoUsuario) {

      usuario.set(nuevoUsuario);
    }

    function esAdmin(x){

      admin.set(x);
    }

  </script>
  
  <!-- Aquí va el marcado del componente -->
  <link
    href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
    rel="stylesheet"
    integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH"
    crossorigin="anonymous"
  />
  <section class="gradient-custom">
    <div class="container py-5 h-100">
      <div class="row d-flex justify-content-center align-items-center h-100">
        <div class="col-12 col-md-8 col-lg-6 col-xl-5">
          <div class="card bg-black text-white" style="border-radius: 1rem;">
            <div class="card-body text-center">
              <div class="mb-md-5 mt-md-4">
                <img
                  src="https://firebasestorage.googleapis.com/v0/b/omegaproxy-4abfe.appspot.com/o/isotipo.png?alt=media&token=55b51f65-c906-4427-ba2b-39bfea66ada9"
                  class="rounded mx-auto d-block"
                  alt="..."
                  width="180px"
                  height="180px"
                />
                <h2 class="fw-bold mb-2 text-uppercase">Inicio de sesión</h2>
                <p class="text-white-50 mb-5">
                  {mensaje}
                </p>
  
                <div data-mdb-input-init class="form-outline form-white mb-4">
                  <input
                    type="email"
                    id="typeEmailX"
                    class="form-control form-control-lg"
                    bind:value={username}
                  />
                  <label class="form-label" for="typeEmailX"
                    >Nombre de usuario</label
                  >
                </div>
  
                <div data-mdb-input-init class="form-outline form-white mb-4">
                  <input
                    type="password"
                    id="typePasswordX"
                    class="form-control form-control-lg"
                    bind:value={password}
                  />
                  <label class="form-label" for="typePasswordX">Contraseña</label>
                </div>
  
                <button
                  on:click={login}
                  data-mdb-button-init
                  data-mdb-ripple-init
                  class="btn btn-outline-light btn-lg px-5"
                  type="submit">Iniciar sesión</button
                >
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
  
  <style>
    .gradient-custom {
      /* fallback for old browsers */
      background: rgba(83, 67, 0, 0);
  
      /* Chrome 10-25, Safari 5.1-6 */
      background: -webkit-linear-gradient(
        to right,
        rgba(65, 97, 174),
        rgba(158, 31, 100)
      );
  
      /* W3C, IE 90+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
      background: linear-gradient(
        to right,
        rgba(65, 97, 174),
        rgba(158, 31, 100)
      );
    }
  </style>
  