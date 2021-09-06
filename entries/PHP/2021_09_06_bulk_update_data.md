---
title: bulk_update_data
category: PHP
date: 2021-09-06
---

function updateMultiple(array $values, string $index = 'id'): bool
    {
        $final = [];
        $ids = [];

        if (\count($values) === 0) {
            return false;
        }
        $bindings = [$index];

        foreach ($values as $key => $val) {
            $ids[] = $val[$index];
            foreach (array_keys($val) as $field) {
                if ($field !== $index) {
                    $value = ($val[$field] === null ? 'NULL' : '"' . $val[$field] . '"');

                    $final[$field][] = 'WHEN `' . $index . '` = "' . $val[$index] . '" THEN ' . $value . ' ';
                }
            }
        }

        $cases = '';
        foreach ($final as $key => $value) {
            $cases .= '`' . $key . '` = (CASE ' . implode("\n", $value) . "\n" . 'ELSE `' . $key . '` END), ';
        }

        $query = "UPDATE `{$this->getTable()}` SET " . substr($cases, 0, -2) . " WHERE `$index` IN(" . implode(',', $ids) . ");";

        logger()->debug('QUERY', compact('query'));
        return DB::statement($query, $bindings);
    }

Refs:
Thanks to @barryvdh
- https://github.com/laravel/ideas/issues/575#issuecomment-300731748