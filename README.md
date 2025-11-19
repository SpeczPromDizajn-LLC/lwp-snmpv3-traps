# lwp-traps-v3

Реализация SNMPv3 Trap на базе стека lwIP

Модуль основан на обсуждении из этой ветки:
https://www.mail-archive.com/lwip-users@nongnu.org/msg20283.html

Но в нём исправлено много ошибок, в том числе проблема с добавлением лишнего 0 в OID.
Я писал об этом разработчикам, но на моё письмо никто не ответил. Ниже привожу его полный текст:

### Description

Hello lwip developers,

I found a bug in snmp_traps.c where an extra zero is incorrectly added 
to enterprise-specific trap OIDs.

### Problem Description

In function snmp_prepare_trap_oid(), for SNMP_GENTRAP_ENTERPRISE_SPECIFIC traps,
the code adds a zero between the enterprise OID and specific trap number:

Current (wrong): enterprise.0.specific-trap
Correct (RFC 3418): enterprise.specific-trap

This results in invalid snmpTrapOID values like:
1.3.6.1.4.1.53722.121.1.0.1 instead of correct 1.3.6.1.4.1.53722.121.1.1

### Proposed Fix

Remove the extra zero insertion for enterprise-specific traps.

## Testing

The fix has been tested to ensure enterprise-specific traps now generate
correct OIDs according to RFC 3418 specification.

## Change Details

Modified file: `src/apps/snmp/snmp_traps.c`
- Removed the extra zero insertion for enterprise-specific traps
- Adjusted length check accordingly

```
diff --git a/src/apps/snmp/snmp_traps.c b/src/apps/snmp/snmp_traps.c
index 1234567..890abcd 100644
--- a/src/apps/snmp/snmp_traps.c
+++ b/src/apps/snmp/snmp_traps.c
@@ -XX,YY +XX,YY @@ snmp_prepare_trap_oid(struct snmp_obj_id *dest_snmp_trap_oid, const struct snmp_
     } else {
       MEMCPY(dest_snmp_trap_oid, eoid, sizeof(*dest_snmp_trap_oid));
     }
-    if (dest_snmp_trap_oid->len + 2 < SNMP_MAX_OBJ_ID_LEN) {
-      dest_snmp_trap_oid->id[dest_snmp_trap_oid->len++] = 0;
+    if (dest_snmp_trap_oid->len + 1 < SNMP_MAX_OBJ_ID_LEN) {
       dest_snmp_trap_oid->id[dest_snmp_trap_oid->len++] = specific_trap;
     } else {
       err = ERR_MEM;
```

---

# Description in English

SNMPv3 Trap implementation based on the lwIP stack

The module is based on the discussion from this thread.:
https://www.mail-archive.com/lwip-users@nongnu.org/msg20283.html

But it fixed a lot of bugs, including the problem with adding an extra 0 to the OID.
I wrote to the developers about this, but no one answered my letter. The full text of the letter is attached above.
