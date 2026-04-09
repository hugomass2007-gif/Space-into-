
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Shoot'em up score progressif</title>
<style>
body{margin:0;overflow:hidden;background:black;}
canvas{display:block;margin:auto;background:black;}
</style>
</head>
<body>
<canvas id="gameCanvas" width="800" height="600"></canvas>
<script>
const canvas=document.getElementById("gameCanvas");
const ctx=canvas.getContext("2d");

const LARGEUR_ECRAN=800;
const HAUTEUR_ECRAN=600;

let score=0;
let gameOver=false;
let message="";
let messageTimer=0;
let gameOverPlayed=false;

// Progression basée sur score
let speedMultiplier=0;  // augmentera tous les 200 points
let spawnMultiplier=0;

// Images
const imgVaisseau=new Image(); imgVaisseau.src="ressource/VAI.png";
const imgEnnemi=new Image(); imgEnnemi.src="ressource/VA.png";
const imgObstacle=new Image(); imgObstacle.src="ressource/OP.png";
const imgMissile=new Image(); imgMissile.src="ressource/Missile_trump.png";
const imgBonus=new Image(); imgBonus.src="ressource/G.png";
const imgExplosion=new Image(); imgExplosion.src="ressource/R.png";

// Sons
const musiqueFond=new Audio("ressource/C.MP3"); musiqueFond.loop=true; musiqueFond.volume=0.5;
let musiqueStart=false;
document.addEventListener("keydown",()=>{
    if(!musiqueStart){musiqueFond.play(); musiqueStart=true;}
});

// Classes
class Vaisseau{
    constructor(){
        this.image=imgVaisseau; this.width=60; this.height=60;
        this.x=LARGEUR_ECRAN/2-this.width/2; this.y=HAUTEUR_ECRAN/2-this.height/2;
        this.vitesse=6; this.invincible=false;
    }
    draw(){ ctx.drawImage(this.image,this.x,this.y,this.width,this.height);}
    update(keys){
        if(keys["ArrowUp"]) this.y-=this.vitesse;
        if(keys["ArrowDown"]) this.y+=this.vitesse;
        if(keys["ArrowLeft"]) this.x-=this.vitesse;
        if(keys["ArrowRight"]) this.x+=this.vitesse;
        // Limites écran
        if(this.x<0) this.x=0;
        if(this.y<0) this.y=0;
        if(this.x+this.width>LARGEUR_ECRAN) this.x=LARGEUR_ECRAN-this.width;
        if(this.y+this.height>HAUTEUR_ECRAN) this.y=HAUTEUR_ECRAN-this.height;
    }
}

class Ennemi{
    constructor(){this.image=imgEnnemi; this.width=80; this.height=80;
        this.x=LARGEUR_ECRAN+Math.random()*200;
        this.y=Math.random()*(HAUTEUR_ECRAN-this.height);
        this.vitesse=4+speedMultiplier;}
    draw(){ctx.drawImage(this.image,this.x,this.y,this.width,this.height);}
    update(){this.x-=this.vitesse;}
}

class Obstacle{
    constructor(){this.image=imgObstacle; this.width=70; this.height=70;
        this.x=LARGEUR_ECRAN+Math.random()*200;
        this.y=Math.random()*(HAUTEUR_ECRAN-this.height);
        this.vitesse=5+speedMultiplier;}
    draw(){ctx.drawImage(this.image,this.x,this.y,this.width,this.height);}
    update(){this.x-=this.vitesse;}
}

class Missile{
    constructor(x,y,size="normal"){
        this.image=imgMissile;
        this.width=size==="big"?40:20; this.height=size==="big"?40:20;
        this.x=x; this.y=y; this.vitesse=size==="big"?14:16;
        this.son=new Audio("ressource/yodguard-short-energy-beam-shot-3-482517.wav");
        this.son.volume=0.3; this.son.play();
    }
    draw(){ctx.drawImage(this.image,this.x,this.y,this.width,this.height);}
    update(){this.x+=this.vitesse;}
}

class Bonus{
    constructor(x,y,type){
        this.image=imgBonus; this.width=40; this.height=40;
        this.x=x; this.y=y; this.vitesse=3; this.type=type;}
    draw(){ctx.drawImage(this.image,this.x,this.y,this.width,this.height);}
    update(){this.x-=this.vitesse;}
}

class Explosion{
    constructor(x,y){this.x=x; this.y=y; this.size=80; this.timer=20;}
    draw(){ctx.drawImage(imgExplosion,this.x-this.size/2,this.y-this.size/2,this.size,this.size); this.timer--;}
}

// Variables jeu
const keys={};
const vaisseau=new Vaisseau();
const missiles=[]; const ennemis=[]; const obstacles=[]; const bonuses=[]; const explosions=[];
const bonusTypes=["points","vitesse","invincible","big_missile"];

document.addEventListener("keydown",e=>{
    keys[e.key]=true;
    if(e.key===" " && !gameOver && missiles.length<15)
        missiles.push(new Missile(vaisseau.x+vaisseau.width,vaisseau.y+vaisseau.height/2));
    if(e.key==="r" && gameOver) location.reload();
});
document.addEventListener("keyup",e=>{keys[e.key]=false;});

function collision(a,b){
    return a.x<b.x+b.width && a.x+a.width>b.x && a.y<b.y+b.height && a.y+a.height>b.y;
}

function playSound(src,volume=0.3){ const s=new Audio(src); s.volume=volume; s.play(); }

// Boucle principale
function gameLoop(){
    ctx.clearRect(0,0,LARGEUR_ECRAN,HAUTEUR_ECRAN);

    if(!gameOver){
        // Progression par score
        speedMultiplier=Math.floor(score/200);
        spawnMultiplier=Math.floor(score/200);

        // Spawn avec légère augmentation
        if(Math.random()<(0.015+spawnMultiplier*0.002) && ennemis.length<40) ennemis.push(new Ennemi());
        if(Math.random()<(0.015+spawnMultiplier*0.002) && obstacles.length<30) obstacles.push(new Obstacle());
        if(Math.random()<0.004 && bonuses.length<5)
            bonuses.push(new Bonus(LARGEUR_ECRAN+20,Math.random()*(HAUTEUR_ECRAN-40),bonusTypes[Math.floor(Math.random()*bonusTypes.length)]));

        // Vaisseau
        vaisseau.update(keys); vaisseau.draw();

        // Missiles
        for(let i=missiles.length-1;i>=0;i--){ let m=missiles[i]; m.update(); m.draw();
            if(m.x>LARGEUR_ECRAN) missiles.splice(i,1); }

        // Ennemis
        for(let i=ennemis.length-1;i>=0;i--){ let e=ennemis[i]; e.update(); e.draw();
            if(e.x+e.width<0) ennemis.splice(i,1);
            if(collision(vaisseau,e) && !vaisseau.invincible){ explosions.push(new Explosion(vaisseau.x,vaisseau.y)); playSound("ressource/S.wav"); gameOver=true; }
            for(let j=missiles.length-1;j>=0;j--){ let m=missiles[j]; if(collision(m,e)){
                explosions.push(new Explosion(e.x+40,e.y+40));
                missiles.splice(j,1); ennemis.splice(i,1);
                playSound("ressource/S.wav"); score+=10; break;}}}

        // Obstacles
        for(let i=obstacles.length-1;i>=0;i--){ let o=obstacles[i]; o.update(); o.draw();
            if(o.x+o.width<0) obstacles.splice(i,1);
            if(collision(vaisseau,o) && !vaisseau.invincible){ explosions.push(new Explosion(vaisseau.x,vaisseau.y)); playSound("ressource/S.wav"); gameOver=true; }
            for(let j=missiles.length-1;j>=0;j--){ let m=missiles[j]; if(collision(m,o)){
                explosions.push(new Explosion(o.x+35,o.y+35)); missiles.splice(j,1); obstacles.splice(i,1);
                playSound("ressource/S.wav"); score-=5; message="-5 points obstacle"; messageTimer=100; break;}}}

        // Bonuses
        for(let i=bonuses.length-1;i>=0;i--){ let b=bonuses[i]; b.update(); b.draw();
            if(collision(vaisseau,b)){
                bonuses.splice(i,1); playSound("ressource/universfield-game-bonus-02-294436.wav");
                if(b.type==="points"){score+=20; message="+20 points !"; messageTimer=120;}
                if(b.type==="vitesse"){vaisseau.vitesse+=2; message="Vitesse boostée !"; messageTimer=120;}
                if(b.type==="invincible"){vaisseau.invincible=true; setTimeout(()=>vaisseau.invincible=false,5000); message="Invincibilité !"; messageTimer=120;}
                if(b.type==="big_missile"){ [-20,0,20].forEach(offset=>missiles.push(new Missile(vaisseau.x+vaisseau.width,vaisseau.y+vaisseau.height/2+offset,"big"))); message="Gros missile !"; messageTimer=120;}}
            if(b.x+b.width<0) bonuses.splice(i,1); }

        // Explosions
        for(let i=explosions.length-1;i>=0;i--){ explosions[i].draw(); if(explosions[i].timer<=0) explosions.splice(i,1); }

        // Score et message
        ctx.fillStyle="white"; ctx.font="24px Arial"; ctx.fillText("Score : "+score,10,30);
        if(messageTimer>0){ ctx.fillStyle="yellow"; ctx.font="28px Arial"; ctx.fillText(message,LARGEUR_ECRAN/2-ctx.measureText(message).width/2,50); messageTimer--;}
        
    } else {
        ctx.fillStyle="red"; ctx.font="70px Arial"; ctx.fillText("GAME OVER",220,260);
        ctx.fillStyle="white"; ctx.font="32px Arial"; ctx.fillText("Score : "+score,330,320);
        ctx.fillText("Appuie sur R pour rejouer",230,370);
        if(!gameOverPlayed){ playSound("ressource/universfield-game-over-deep-male-voice-clip-352695.wav",0.5); gameOverPlayed=true;}
    }

    setTimeout(()=>{requestAnimationFrame(gameLoop);},16);
}

gameLoop();
</script>
</body>
</html>
