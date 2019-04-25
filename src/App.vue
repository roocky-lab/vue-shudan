<template>
    <div id="app">
        <Goban></Goban>
        <hr>
        <BoundedGoban></BoundedGoban>
    </div>
</template>

<script>
import { Goban, BoundedGoban } from './components/Shudan';

export default {
    name: 'app',
    components: {
        Goban,
        BoundedGoban
    }
};
</script>
