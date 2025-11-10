--[[ 
 Auto Collector — versión personalizada
 Detecta y recolecta automáticamente modelos con etiqueta "Comun"
 y nombre "Noobini Lusinini" (comparación case-insensitive).
--]]

-- CONFIGURACIÓN
local etiquetaObjetivo = "COMUN"               -- etiqueta a buscar (usar uppercase en comparación)
local nombreObjetivo = "NOOBINI LUSININI"      -- nombre exacto objetivo (uppercase)
local tiempoEntreChecks = 0.5                  -- segundos de espera antes de procesar nuevo child
local nombrePasarela = "Pasarela"              -- carpeta donde aparecen los modelos
local nombreContenedor = "ContenedorOG"        -- destino opcional (puedes cambiarlo)

-- FUNCIONES AUXILIARES
local function nombreEsObjetivo(obj)
    if not obj or not obj:IsA("Model") then return false end
    local nombre = obj.Name or ""
    return string.upper(nombre) == nombreObjetivo
end

local function tieneEtiquetaComun(obj)
    -- Aquí asumimos que usas el nombre del modelo como "etiqueta" o alguna child llamada "Tag".
    -- Ajusta esta función si usas CollectionService, BoolValues, StringValues, etc.
    if not obj or not obj:IsA("Model") then return false end

    -- 1) Si usas CollectionService y etiquetas reales, descomenta la sección correspondiente:
    --[[
    local CollectionService = game:GetService("CollectionService")
    if CollectionService:HasTag(obj, "Comun") then
        return true
    end
    ]]

    -- 2) Si la "etiqueta" está representada por un StringValue llamado "Etiqueta" dentro del modelo:
    local etiquetaChild = obj:FindFirstChild("Etiqueta")
    if etiquetaChild and etiquetaChild:IsA("StringValue") then
        if string.upper(etiquetaChild.Value or "") == etiquetaObjetivo then
            return true
        end
    end

    -- 3) Como fallback, también revisamos si el nombre contiene la palabra "Comun"
    if string.find(string.upper(obj.Name or ""), etiquetaObjetivo) then
        return true
    end

    return false
end

local function esCoincidente(obj)
    return nombreEsObjetivo(obj) and tieneEtiquetaComun(obj)
end

local function recolectar(obj)
    if not obj or not obj:IsDescendantOf(workspace) then return end
    print("[Collector] Recolectando ->", obj.Name)

    local contenedor = workspace:FindFirstChild(nombreContenedor)
    if contenedor and contenedor:IsA("BasePart") then
        local hrp = obj:FindFirstChild("HumanoidRootPart") or obj.PrimaryPart
        if hrp then
            hrp.CFrame = contenedor.CFrame + Vector3.new(0, 3, 0)
            return
        end
    end

    -- Si no hay contenedor o no se puede mover, destruye como fallback
    pcall(function() obj:Destroy() end)
    print("[Collector] Eliminado como fallback:", obj.Name)
end

-- INICIALIZACIÓN
local pasarela = workspace:FindFirstChild(nombrePasarela)
if not pasarela then
    warn("[Collector] No se encontró la pasarela:", nombrePasarela)
    return
end

print("[Collector] Activado. Buscando modelos 'Comun' llamados 'Noobini Lusinini'...")

-- Evento para nuevos hijos
pasarela.ChildAdded:Connect(function(child)
    task.wait(tiempoEntreChecks)
    if esCoincidente(child) then
        recolectar(child)
    end
end)

-- Escaneo inicial
for _, child in ipairs(pasarela:GetChildren()) do
    if esCoincidente(child) then
        recolectar(child)
    end
end
