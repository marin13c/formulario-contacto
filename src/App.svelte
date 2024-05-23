<script>
  // Importa la función onMount de Svelte para ejecutar código cuando el componente se monta
  import { onMount } from "svelte";
  import servicesData from "./JSONFRONTEND.json";

  // Define un objeto registers para almacenar los datos del formulario
  let registers = {
    name: "",
    last_name: "",
    email: "",
    phone: "",
    message: "",
    services: [],
  };

  let selectedCategory = []; // Categorías seleccionadas para filtrar servicios
  let filteredServices = []; // Servicios filtrados según las categorías seleccionadas
  let showModal = false; // Estado para mostrar u ocultar el modal
  let successMessage = false; // Estado para mostrar el mensaje de éxito
  let isServicesEmpty = registers.services.length === 0; // Verifica si no hay servicios seleccionados
  let selectedCategories = new Set(); // Conjunto de categorías seleccionadas
  let hoveredService = null; // Servicio actualmente bajo el cursor (hover)

  // Función para agregar o quitar un servicio de la lista de servicios seleccionados
  const toggleService = (service) => {
    if (registers.services.includes(service.nombre)) {
      // Si el servicio ya está seleccionado, lo elimina de la lista
      registers.services = registers.services.filter(
        (s) => s !== service.nombre
      );
    } else {
      // Si el servicio no está seleccionado, lo agrega a la lista
      registers.services = [...registers.services, service.nombre];
    }
  };

  // Función para agregar o quitar una categoría de la lista de categorías seleccionadas
  const toggleCategory = (category) => {
    if (selectedCategories.has(category)) {
      selectedCategories.delete(category);
    } else {
      selectedCategories.add(category);
    }
    filterByCategory(); // Filtra los servicios según las categorías seleccionadas
  };

  // Función para filtrar servicios según las categorías seleccionadas
  const filterByCategory = () => {
    filteredServices = servicesData.servicios.filter((service) => {
      if (selectedCategory.length === 0) return true; // Si no hay categorías seleccionadas, muestra todos los servicios
      return (
        selectedCategory.includes("") ||
        selectedCategory.some((category) =>
          service.categorias.includes(category)
        )
      );
    });
  };

  // Ejecuta esta función cuando el componente se monta
  onMount(() => {
    registers.services = [];
    filterByCategory(); // Filtra los servicios al montar el componente
  });

  // Función para manejar el envío del formulario
  const onSubmitHandler = async (e) => {
    e.preventDefault(); // Previene el comportamiento predeterminado del formulario
    console.log(registers);

    // Verifica si hay campos vacíos
    const isEmpty = Object.values(registers).some((val) => val === "");
    const isServicesEmpty = registers.services.length === 0;

    if (isEmpty || isServicesEmpty) {
      successMessage = false; // No muestra el mensaje de éxito
      showModal = true; // Muestra el modal de error
      return;
    }
    try {
      const response = await fetch("url", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(registers),
      });

      if (response.ok) {
        console.log("Data successfully sent");
        showModal = true;
        successMessage = true; // Muestra el mensaje de éxito
      } else {
        console.error("Error sending data");
      }
    } catch (error) {
      console.error("Error sending data:", error);
    }
  };

  // Función para cerrar el modal
  const handleCloseModal = () => {
    showModal = false;
    if (successMessage) {
      window.location.reload(); // Recarga la página si el mensaje de éxito se muestra
    }
  };

  // Genera una lista única de categorías a partir de los datos de servicios
  const uniqueCategories = Array.from(
    new Set(servicesData.servicios.flatMap((service) => service.categorias))
  );

  // Función para mostrar/ocultar el menú móvil
  function toggleMobileMenu() {
    const mobileMenu = document.getElementById("mobile-menu");
    const menuButton = document.querySelector(
      'button[aria-controls="mobile-menu"]'
    );
    const expanded = menuButton.getAttribute("aria-expanded") === "true";

    if (expanded) {
      mobileMenu.classList.remove("block");
      mobileMenu.classList.add("hidden");
      menuButton.setAttribute("aria-expanded", "false");
    } else {
      mobileMenu.classList.remove("hidden");
      mobileMenu.classList.add("block");
      menuButton.setAttribute("aria-expanded", "true");
    }
  }
</script>

{#if showModal}
  <!-- Modal -->
  <div
    class="fixed top-0 left-0 w-full h-full flex items-center justify-center bg-black bg-opacity-50"
  >
    <!-- Contenedor del contenido del modal -->
    <div class="bg-white p-6 rounded shadow-lg">
      <h2 class="text-lg font-bold mb-4">Mensaje</h2>

      <!-- Mensajes condicionados -->
      {#if successMessage}
        <p class="mb-4">
          The data has been sent correctly We will be contacting you soon!
        </p>
      {:else if isServicesEmpty}
        <p class="mb-4">Please select at least one service!</p>
      {:else}
        <p class="mb-4">Please complete all fields!</p>
      {/if}
      <button
        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
        on:click={handleCloseModal}
      >
        OK
      </button>
    </div>
  </div>
{/if}

<nav>
  <div class="mx-auto max-w-7xl px-2 sm:px-6 lg:px-8">
    <div class="relative flex h-16 items-center justify-between">
      <div
        class="flex flex-1 items-center justify-center sm:items-stretch sm:justify-start"
      >
        <!-- Botón para el menú móvil -->
        <button
          type="button"
          class="inline-flex items-center justify-center rounded-md p-2 text-gray-400 hover:bg-gray-700 hover:text-white focus:outline-none focus:ring-2 focus:ring-inset focus:ring-white sm:hidden"
          aria-controls="mobile-menu"
          aria-expanded="false"
          on:click={toggleMobileMenu}
        >
          <span class="sr-only">Open main menu</span>
          <!-- Icono de menú (tres barras) -->
          <svg
            class="block h-6 w-6"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
            aria-hidden="true"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M4 6h16M4 12h16M4 18h16"
            ></path>
          </svg>
          <!-- Icono de cerrar menú (X) -->
          <svg
            class="hidden h-6 w-6"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
            aria-hidden="true"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M6 18L18 6M6 6l12 12"
            ></path>
          </svg>
        </button>
        <!-- Menú principal en pantallas grandes -->
        <div class="hidden sm:ml-6 sm:block">
          <div class="flex space-x-4">
            <a
              href="#"
              class="text-white hover:text-gray-500 rounded-md px-3 py-2 text-sm font-medium"
              aria-current="page">Home</a
            >
            <a
              href="#"
              class="text-white hover:text-gray-500 rounded-md px-3 py-2 text-sm font-medium flex items-center"
            >
              Services
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-4 w-4 ml-1"
                viewBox="0 0 20 20"
                fill="#ffffff"
              >
                <path d="M10 12l-5-5h10l-5 5z"></path>
              </svg>
            </a>
            <a
              href="#"
              class="text-white hover:text-gray-500 rounded-md px-3 py-2 text-sm font-medium flex items-center"
            >
              About Us
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-4 w-4 ml-1"
                viewBox="0 0 20 20"
                fill="#ffffff"
              >
                <path d="M10 12l-5-5h10l-5 5z"></path>
              </svg>
            </a>
            <a
              href="#"
              class="text-white hover:text-gray-500 rounded-md px-3 py-2 text-sm font-medium"
              >Pricing</a
            >
          </div>
        </div>
      </div>
      <!-- Botón Apply -->
      <div
        class="absolute inset-y-0 right-0 flex items-center pr-2 sm:static sm:inset-auto sm:ml-6 sm:pr-0"
      >
        <li>
          <a
            href="#"
            class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
            >Apply</a
          >
        </li>
      </div>
    </div>
  </div>

  <!-- Menú móvil -->
  <div class="sm:hidden hidden" id="mobile-menu">
    <div class="px-2 pt-2 pb-3 space-y-1">
      <a
        href="#"
        class="text-gray-300 hover:bg-gray-700 hover:text-white block px-3 py-2 rounded-md text-base font-medium"
        >Home</a
      >
      <a
        href="#"
        class="text-gray-300 hover:bg-gray-700 hover:text-white block px-3 py-2 rounded-md text-base font-medium"
        >Services</a
      >
      <a
        href="#"
        class="text-gray-300 hover:bg-gray-700 hover:text-white block px-3 py-2 rounded-md text-base font-medium"
        >About Us</a
      >
      <a
        href="#"
        class="text-gray-300 hover:bg-gray-700 hover:text-white block px-3 py-2 rounded-md text-base font-medium"
        >Pricing</a
      >
    </div>
  </div>
</nav>

<main
  class="container mx-auto mt-4 flex flex-col items-center px-4 sm:px-6 lg:px-8"
>
  <form
    class="shadow-md rounded px-4 sm:px-10 pt-6 pb-8 mb-4 w-full sm:w-3/4 lg:w-1/2 bg-gradient mx-auto"
    on:submit={onSubmitHandler}
  >
    <div
      class="text-3xl sm:text-4xl lg:text-6xl font-semibold leading-tight mb-4 font-bb-casual-pro-medium text-gradient"
      style="text-align: left;"
    >
      Contact Us
    </div>
    <div
      class="text-xl sm:text-2xl lg:text-4xl font-semibold leading-tight mb-4 text-white font-bb-casual-pro-medium"
      style="text-align: left"
    >
      Let us know what you need
    </div>
    <div class="leading-tight mb-4 text-white" style="text-align: left">
      Please fill all these spaces, for us to reach you with the best quote.
    </div>
    <div class="mb-4 flex flex-col sm:flex-row">
      <div class="w-full sm:w-3/5 sm:pr-2 mb-4 sm:mb-0">
        <!-- Campo para el nombre -->
        <label class="block text-white text-sm font-bold mb-2" for="name"
          >First Name</label
        >
        <input
          class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          bind:value={registers.name}
          type="text"
          id="name"
        />
      </div>
      <div class="w-full sm:w-2/5 sm:pl-2">
        <!-- Campo para el apellido -->
        <label class="block text-white text-sm font-bold mb-2" for="last_name"
          >Last Name</label
        >
        <input
          class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          bind:value={registers.last_name}
          type="text"
          id="last_name"
        />
      </div>
    </div>
    <div class="mb-4">
      <!-- Campo para el correo electrónico -->
      <label class="block text-white text-sm font-bold mb-2" for="email"
        >Email</label
      >
      <input
        class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
        bind:value={registers.email}
        type="text"
        id="email"
      />
    </div>
    <div class="mb-4">
      <!-- Campo para el teléfono -->
      <label class="block text-white text-sm font-bold mb-2" for="phone"
        >Phone</label
      >
      <input
        class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
        bind:value={registers.phone}
        type="text"
        id="phone"
      />
    </div>
    <div class="mb-4">
      <!-- Campo para el mensaje -->
      <label class="block text-white text-sm font-bold mb-2" for="message"
        >Message</label
      >
      <textarea
        class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
        bind:value={registers.message}
        id="message"
        rows="5"
      ></textarea>
    </div>
    <div class="mb-4">
      <!-- Selección de servicios -->
      <label class="block text-white text-sm font-bold mb-2" for="services"
        >Services</label
      >
      <select
        class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
        bind:value={selectedCategory}
        multiple
        on:change={toggleCategory}
      >
        <option value="">All Categories</option>
        {#each uniqueCategories as category}
          <option value={category}>{category}</option>
        {/each}
      </select>
    </div>
    <div class="mb-4">
      <!-- Lista de servicios filtrados con checkbox -->
      {#each filteredServices as service}
        <div class="mb-2 relative">
          <input
            type="checkbox"
            id={service.nombre}
            checked={registers.services.includes(service.nombre)}
            on:change={() => toggleService(service)}
            class="mr-2 leading-tight"
          />
          <label
            for={service.nombre}
            class="text-white text-sm"
            on:mouseenter={() => (hoveredService = service)}
            on:mouseleave={() => (hoveredService = null)}
          >
            {service.nombre}
          </label>
          <!-- Tooltip que muestra la descripción y precio del servicio -->
          {#if hoveredService && hoveredService.nombre === service.nombre}
            <div
              class="absolute bg-white text-black text-sm p-2 rounded shadow-lg"
              style="left: 15rem; top: 0;"
            >
              {hoveredService.descripcion} ({hoveredService.precio})
            </div>
          {/if}
        </div>
      {/each}
    </div>
    <!-- Botón para enviar el formulario -->
    <div class="flex items-center justify-between mt-4">
      <button
        class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline w-full"
        type="submit"
      >
        Get started
      </button>
    </div>
  </form>
</main>

<style lang="postcss">
  :global(html) {
    background-color: #021325;
  }

  .font-bb-casual-pro-medium {
    font-family: "BB Casual Pro Medium", sans-serif;
  }

  .text-gradient {
    background: linear-gradient(0deg, #0683ff, rgba(0, 128, 255, 0.84)),
      linear-gradient(270deg, #0683ff 14.19%, #065aaf 100.84%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent !important;
  }
</style>
