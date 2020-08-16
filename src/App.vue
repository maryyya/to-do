<template>
    <v-app>
        <v-app-bar app dark color="indigo">
            <v-app-bar-nav-icon>
                <v-icon large color="darken-2">mdi-list-status</v-icon>
            </v-app-bar-nav-icon>
            <v-toolbar-title>My To Do's</v-toolbar-title>
            <div class="d-flex align-center"></div>
        </v-app-bar>

        <v-main>
            <Tasks/>
        </v-main>
    </v-app>
</template>

<script>
    // import HelloWorld from './components/HelloWorld';
    import Tasks from './components/Tasks';

    export default {
        name: 'App',

        components: {
            Tasks,
        },

        data: () => ({
            //
        }),
    };
</script>
