const CACHE='estore-v24';
const HOME='./index.html';
self.addEventListener('install',event=>{
  self.skipWaiting();
  event.waitUntil((async()=>{
    const cache=await caches.open(CACHE);
    try{const r=await fetch(HOME,{cache:'no-store'});if(r.ok)await cache.put(HOME,r.clone())}catch(e){}
  })());
});
self.addEventListener('activate',event=>{
  event.waitUntil((async()=>{
    const keys=await caches.keys();
    await Promise.all(keys.filter(k=>k!==CACHE).map(k=>caches.delete(k)));
    await self.clients.claim();
  })());
});
self.addEventListener('fetch',event=>{
  if(event.request.method!=='GET')return;
  const req=event.request;
  event.respondWith((async()=>{
    try{
      const fresh=await fetch(req,{cache:'no-store'});
      if(fresh && fresh.ok){
        const cache=await caches.open(CACHE);
        cache.put(req,fresh.clone()).catch(()=>{});
      }
      return fresh;
    }catch(e){
      const cached=await caches.match(req,{ignoreSearch:true});
      if(cached)return cached;
      if(req.mode==='navigate'){
        const home=await caches.match(HOME,{ignoreSearch:true});
        if(home)return home;
      }
      throw e;
    }
  })());
});
