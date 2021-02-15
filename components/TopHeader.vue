<template>
    <b-navbar toggleable="lg" type="dark" variant="info">
    <b-navbar-brand href="/">DESTACAMEBUS</b-navbar-brand>

    <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

    <b-collapse id="nav-collapse" is-nav>
      <b-navbar-nav>
        <b-nav-item to="/trayectos">Trayectos</b-nav-item>
        <b-nav-item to="/choferes">Choferes</b-nav-item>
        <b-nav-item to="/buses" >Buses</b-nav-item>
        <b-nav-item to="/pasajeros">Pasajeros</b-nav-item>
      </b-navbar-nav>
    </b-collapse>
  </b-navbar>
</template>