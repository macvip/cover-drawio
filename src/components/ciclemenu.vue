<template>
    <nav class="circle-navigation">
        <button class="circle-navigation__nav-btn" @click="toggleMenu" :style="{backgroundImage: menu_btn.icon}"></button>
        <ul class="circle-navigation__list" @click="myClick">
            <li class="circle-navigation__item"
                    v-for="item, i in menu_items"
                    :key="item.name"
                    :style="{transform: item.position, transitionDelay: setURL[i].delay, opacity: isOpenMenu? 0.7:0}"
                    >
                <div :name="item.name" class="circle-navigation__link" :style="{backgroundImage: setURL[i].icon}"></div>
            </li>
        </ul>
    </nav>
</template>
<script>
const {sin, cos, floor, PI} = Math

export default {
    name: 'CicleMenu',
    inject: ['itemClicked'],
data() {
    return {
        initialDistans: 0,
        distanceBeforeItems: 60,
        delayAnimation: 30,
        isOpenMenu: false,
        maxRadius: PI,
        menu_btn: {
            name: 'menu',
            icon: 'url(images/toolbar/menu.png)'
        },
        menu_items: [
            {
                name: 'Preview',
                icon:'preview.png',
                position: ''
            },
            {
                name: 'Refresh',
                icon: 'refresh.png',
                position: ''
            },
            {
                name: 'Edit',
                icon: 'edit.png',
                position: ''
            },
            {
                name: 'Save',
                icon: 'png.png',
                position: ''
            },
            {
                name: 'Copy',
                icon: 'copy.png',
                position: ''
            },
            
        ],
    }
},

    methods: {
        toggleMenu(){
            if(!this.isOpenMenu){
                this.initialDistans = this.distanceBeforeItems;
            }else{
                this.initialDistans = 0;
            }
            let items_amount = this.menu_items.length;
            let angle = (this.maxRadius / items_amount) + (18*PI/360);
            this.menu_items = this.menu_items.map((v, i)=>{
                return {
                    ...v,
                    position: `translate(${floor(sin(i * angle + PI/2)*this.initialDistans)}px, ${floor(cos(i * angle + PI/2)*-this.initialDistans)}px)`,
                }
            });
            this.isOpenMenu = !this.isOpenMenu;
        },
        myClick(e) {
            this.itemClicked(e.target.getAttribute('name'))
            this.isOpenMenu = false
        }
    },
    computed: {
        setURL(){
            return this.menu_items.map((v, i)=>{
                return {...v, icon: `url(${'images/toolbar/' + v.icon})`, delay: `${i / this.delayAnimation}s`};
            });
        }
    },
}

</script>
<style lang="scss" scoped>
$accent: #ffcc80;
.circle-navigation{
/*  position: absolute;
    top: 50px;
    right: 40px;
    transform: translate(-50%, -50%);
    z-index: 1;
*/
    &__nav-btn{
        width: 48px;
        height: 48px;
        border-radius: 50%;
        border: none;
        transition: .3s ease;
        cursor: pointer;
        background-repeat: no-repeat;
        background-position: center;
        background-color: #ffe0b2;
        background-size: 32px;
        z-index: 15;
        outline: none;
        opacity: 0.55;
        &:hover{
            background: $accent;
            background-repeat: no-repeat;
            background-position: center;
            background-size: 36px;
            box-shadow: 1px 2px 10px #ffb74d;
            opacity: 0.7;
        }
        
    }
    &__list{
        list-style-type: none;
        padding: 0;
        margin: 0;
        position: absolute;
        top: 4px;
        left: 4px;
        z-index: -1;
    }
    &__item{
        position: absolute;
        transition-property: all;
        transition-timing-function: cubic-bezier(0.86, 0, 0.07, 1);
        transition-duration: .2s;
        border-radius: 50%;
        background-color: #ffe0b2;
    }
    &__link{
        color: #000;
        text-decoration: none;
        display:block;
        width: 40px;
        height: 40px;
        border-radius: 50%;
        background-size: 32px;
        background-repeat: no-repeat;
        background-position: center;
        &:hover{
            background-size: 36px!important;
            background-repeat: no-repeat!important;
            background-position: center!important;
            transition: .3s ease;
            background: $accent;
            box-shadow: 1px 2px 10px #ffb74d;
        }
    }
}
.icon-position{
    background-repeat: no-repeat;
    background-position: center;
}
</style>
